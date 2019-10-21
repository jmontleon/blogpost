How to Fix EL8 Golang Execshield Issues
=======================================

Introduction
------------
Recently my team started working on bring upstream work downstream. When performing an RPM build of one of the golang projects it was flagged with an Execshield issue. Specifically, it was marked as, "FAIL: Entry point instruction is not ENDBR64". This originally didn't mean much to me, but I went off to do some reading. 

What is the ENDBR64 instruction
-------------------------------
After some searching and reading I discovered that the ENDBR64 instruction is a feature introduced in Intel's [Control-flow Enforcement Technology](https://software.intel.com/sites/default/files/managed/4d/2a/control-flow-enforcement-technology-preview.pdf) (CET) to help prevent [Return-oriented programming](https://en.wikipedia.org/wiki/Return-oriented_programming) (ROP) exploits. From Intel's documentation I learned, "ENDBRANCH is a new instruction that isused to mark valid jump target addresses of indirect calls and jumps in the program",  and is part of their CET Indirect Branch Tracking (IBT) capability.

How can I check a binary for myself
-----------------------------------
Additional searches brought me to a helpful [bug](https://bugzilla.redhat.com/show_bug.cgi?id=1652925). Here I learned I could do the following:

`readelf -hW /path/to/binary | grep Entry`

Then with the return value:

`objdump -d --reloc /path/to/binary | grep -E '^\s+<value>:'`

Confirming the problem:
-----------------------
I started with a basic Hello World application, figuring this would be the easiest way to go about testing.

```
$ cat hello-world.go 
package main

import "fmt"

func main() {
	fmt.Println("hello world")
}

$ go build hello-world.go

$ readelf -hW hello-world | grep Entry
  Entry point address:               0x454ae0

$ objdump -d --reloc hello-world | grep -E '^\s+454ae0:'
  454ae0:	e9 2b c6 ff ff       	jmpq   451110 <_rt0_amd64>
```

This is what I expected to see. Our program does not contain the endbr64 instruction.

First attempt to enable IBT
---------------------------
My searches also brought up many links about CET and IBT being enabled in gcc. Knowing that there are two go compilers, gc the compiler go tool uses by default, and gcc-go I decided to give gcc-go a try and went ahead and installed it.

`$ sudo dnf install gcc-go`

```
$ go build hello-world.go

$ readelf -hW hello-world | grep Entry
  Entry point address:               0x402430

$ objdump -d --reloc hello-world | grep -E '^\s+402430'
  402430:	f3 0f 1e fa          	endbr64 
```

This looked great. And I thought I was ready to go until I attempted to build our project with gcc-go.
```
$ go build
# github.com/fusor/cpma
/usr/bin/ld: /home/jmontleo/.cache/go-build/7b/7b263e49c5cca276fddfc35a9b07489eb311de98ce3f9d590a26decec516db10-d(_go_.o): in function `github.com..z2ffusor..z2fcpma..z2fvendor..z2fgithub.com..z2fmodern..z2dgo..z2freflect2.loadGo15Types':
/home/jmontleo/Documents/openshift/src/github.com/fusor/cpma/vendor/github.com/modern-go/reflect2/type_map.go:42: undefined reference to `reflect.typelinks'
/usr/bin/ld: /home/jmontleo/.cache/go-build/7b/7b263e49c5cca276fddfc35a9b07489eb311de98ce3f9d590a26decec516db10-d(_go_.o): in function `reflect2.loadGo17Types':
/home/jmontleo/Documents/openshift/src/github.com/fusor/cpma/vendor/github.com/modern-go/reflect2/type_map.go:74: undefined reference to `reflect.typelinks'
/usr/bin/ld: /home/jmontleo/Documents/openshift/src/github.com/fusor/cpma/vendor/github.com/modern-go/reflect2/type_map.go:78: undefined reference to `reflect.resolveTypeOff'
collect2: error: ld returned 1 exit status
```

After a little digging I discovered a [mailing list reply](https://www.mail-archive.com/golang-nuts@googlegroups.com/msg31266.html) about building etcd with llvm. Here I discovered modern-go, one of our dependencies, cannot be built with gcc-go. At this point I thought we were stuck and we'd have to see if we could obtain an exception.

I removed gcc-go
```
sudo dnf remove -y gcc-go
```

Looking at etcd on Fedora
-------------------------
Out of curiosity I took a look at the etcd binary from the Fedora etcd package.

```
$ readelf -hW /usr/bin/etcd | grep Entry
  Entry point address:               0x5c72c0

$ objdump -d --reloc /usr/bin/etcd | grep -E '^\s+5c72c0'
  5c72c0:	f3 0f 1e fa          	endbr64 
```

So there must be another way.

Second attempt to enable IBT
---------------------------
I decided to look at the Fedora sources to see how they were building etcd in the RPM spec.

Here they are using an RPM macro to do the build:
https://src.fedoraproject.org/rpms/etcd/blob/f31/f/etcd.spec#_133

I went ahead and evaluated the macro on Fedora 31:
```
$ rpm --eval %gobuild

  # https://bugzilla.redhat.com/show_bug.cgi?id=995136#c12
    %ifnarch ppc64
   GO111MODULE=off \
  go build -buildmode pie -compiler gc -tags="rpm_crashtraceback ${BUILDTAGS:-}" -ldflags "${LDFLAGS:-} -B 0x$(head -c20 /dev/urandom|od -An -tx1|tr -d ' \n') -extldflags '-Wl,-z,relro -Wl,--as-needed  -Wl,-z,now -specs=/usr/lib/rpm/redhat/redhat-hardened-ld '" -a -v -x ;
  %else
   GO111MODULE=off \
  go build                -compiler gc -tags="rpm_crashtraceback ${BUILDTAGS:-}" -ldflags "${LDFLAGS:-} -B 0x$(head -c20 /dev/urandom|od -An -tx1|tr -d ' \n') -extldflags '-Wl,-z,relro -Wl,--as-needed  -Wl,-z,now -specs=/usr/lib/rpm/redhat/redhat-hardened-ld '" -a -v -x ;
  %endif
```

Let's try this on our hello-world binary:
```
$ go build -buildmode pie -compiler gc -tags="rpm_crashtraceback ${BUILDTAGS:-}" -ldflags "${LDFLAGS:-} -B 0x$(head -c20 /dev/urandom|od -An -tx1|tr -d ' \n') -extldflags '-Wl,-z,relro -Wl,--as-needed  -Wl,-z,now -specs=/usr/lib/rpm/redhat/redhat-hardened-ld '" -a hello-world.go

$ readelf -hW hello-world | grep Entry
  Entry point address:               0x76220

$ objdump -d --reloc hello-world | grep -E '^\s+76220'
   76220:	f3 0f 1e fa          	endbr64 
```

I decided to make a quick attempt at fixing our RPM with by replacing 'go build' with '%gobuild' in our RPM and attempting a rebuild, but the build failed.

Third attempt to enable IBT, once more with feeling
---------------------------------------------------
On RHEL 8 %gobuild does not evaluate. More digging revealed that this macro is provided by the `go-rpm-macros` on Fedora. This package does not exist on RHEL 8, but this isn't a major issue for us. We just need a few more lines in the RPM spec to define the macro.

```
%if ! 0%{?gobuild:1}
%define gobuild(o:) go build -buildmode pie -tags=rpm_crashtraceback -ldflags "${LDFLAGS:-} -B 0x$(head -c20 /dev/urandom|od -An -tx1|tr -d ' \\n') -extldflags '-Wl,-z,relro,-z,now'" -a -v -x %{?**};
%endif
```

Once the macro is defined, the build succeeds, and we find the endbr64 instruction as we did in the hellow-world example.

Changes to our projects RPM spec:
https://github.com/fusor/cpma/commit/441eb4579e4004f1af3a4562f7cf6dfe7e31bf3d
