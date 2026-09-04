# Mojarra 2.2.13 – CVE-2020-6950 Backport

This repository contains a reproducible backport of the fix for **CVE-2020-6950** to **Mojarra 2.2.13**.

It exists primarily for traceability and for maintaining legacy applications which, for compatibility reasons, cannot simply be upgraded to a newer JSF/Mojarra release.

## Credits and upstream sources

### Mojarra

Mojarra is the reference implementation of JavaServer Faces (JSF), originally developed as part of the GlassFish/Java EE ecosystem and now maintained as the Eclipse Mojarra project.

The Mojarra source code contained in this repository is **not my work**. All credit and copyright remain with the original Mojarra authors and contributors.

Current upstream project:

* Eclipse Mojarra: https://github.com/eclipse-ee4j/mojarra
* Eclipse project page: https://projects.eclipse.org/projects/ee4j.mojarra

The unmodified Mojarra 2.2.13 artifacts used as the basis for this work were obtained from Maven Central.

### CVE-2020-6950 patch

The security fix used here was **not developed by me**.

The backport for Mojarra 2.2.13 was obtained from the **openEuler** Mojarra package, where `CVE-2020-6950.patch` is applied to Mojarra 2.2.13.

openEuler package source:

* https://gitee.com/src-openeuler/mojarra

The original patch has been retained so that the origin and exact nature of the security modification remain traceable.

My thanks go to the upstream Mojarra developers and to the openEuler maintainers and contributors who provided and published the backported security patch.

## Why this repository exists

I encountered a legacy application which still depends on Mojarra 2.2.13 and was affected by CVE-2020-6950.

Upgrading the complete JSF implementation was not a practical short-term option because the application and its surrounding framework are legacy software with significant compatibility constraints.

At the same time, I wanted the security modification to be:

* based on a publicly available upstream/vendor patch,
* minimal,
* reviewable,
* reproducible,
* and traceable back to the original Mojarra 2.2.13 implementation.

For that reason I rebuilt the affected classes myself rather than distributing an unexplained modified binary.

This repository therefore does not represent an independent implementation of the security fix. It documents and packages an existing published fix for use with Mojarra 2.2.13.

## What was changed

The openEuler CVE-2020-6950 patch modifies three Mojarra classes:

```text id="3vfwg8"
com/sun/faces/application/resource/ClasspathResourceHelper.java
com/sun/faces/application/resource/ResourceManager.java
com/sun/faces/application/resource/WebappResourceHelper.java
```

The patch adds validation of the resource contract name using:

```text id="gjx2gd"
ResourceManager.nameContainsForbiddenSequence(...)
```

Only these three classes are replaced in the patched `jsf-impl-2.2.13` JAR.

A comparison of the unpacked original and patched JAR was performed to verify that no other files differ.

## Build approach

The original Mojarra 2.2.13 sources were used together with the published openEuler CVE patch.

Because reconstructing the complete historical Mojarra 2.2.13 build environment and its old build dependencies would add unnecessary complexity and make the result harder to audit, only the three affected classes were recompiled.

They were compiled with OpenJDK 8 using:

```text id="jhw52d"
-source 1.5 -target 1.5
```

The resulting class files were verified to have:

```text id="hyebwp"
major version: 49
```

which matches the bytecode version of the original Mojarra 2.2.13 classes.

The three resulting class files were then inserted into an otherwise unchanged copy of the original `jsf-impl-2.2.13.jar`.

## Verification

The resulting JAR was unpacked and compared against the original.

Only these files differ:

```text id="y26vkw"
ClasspathResourceHelper.class
ResourceManager.class
WebappResourceHelper.class
```

The resulting bytecode was additionally inspected with `javap` to verify that both resource helper classes contain the intended call to:

```text id="4bj29h"
ResourceManager.nameContainsForbiddenSequence(...)
```

All three replacement classes were also verified to retain Java class-file major version 49.

## Deployment artifact

For applications which currently use the split Mojarra libraries:

```text id="3c9v5d"
jsf-api-2.2.13.jar
jsf-impl-2.2.13.jar
```

the relevant replacement produced by this project is:

```text id="hj90wp"
jsf-impl-2.2.13-patched.jar
```

The JAR filename itself is not significant to the Java class loader. The `-patched` suffix is intentionally retained so that administrators can immediately distinguish the modified implementation from the original one.

**Do not deploy both the original and patched `jsf-impl` JAR simultaneously.**

A safe deployment procedure is to back up the existing application first, remove the original `jsf-impl-2.2.13.jar`, deploy `jsf-impl-2.2.13-patched.jar`, restart the application/container, and verify both normal application operation and the security issue addressed by CVE-2020-6950.

## Scope and disclaimer

This repository is intended as a narrowly scoped compatibility/security backport for legacy Mojarra 2.2.13 installations.

It is **not** a fork of Mojarra, not a new Mojarra release, and is not affiliated with or endorsed by the Eclipse Foundation, Oracle, GlassFish, or openEuler.

Where possible, upgrading to a currently maintained JSF/Jakarta Faces implementation is preferable to maintaining an old Mojarra release with individual security patches.

This repository addresses the specific CVE described above. It should **not** be interpreted as making Mojarra 2.2.13 generally secure, current, maintained, or supported.

Users remain responsible for testing the patched artifact in their own environment before deploying it to production systems.

## Licensing

The original Mojarra source code and binaries remain subject to their respective upstream licenses and copyright notices.

Mojarra 2.2.13 was distributed under the GPL v2 / CDDL dual-license terms applicable to that release, including the GPL Classpath Exception where stated in the original source headers.

The original Mojarra license file is included unchanged at:

```text id="dk9a7q"
packager/legal/LICENSE.txt
```

The original copyright and license headers in the Mojarra source files have been retained.

The openEuler-provided patch remains attributable to its respective authors and source project.

No ownership of the original Mojarra code or the CVE fix is claimed by this repository.
