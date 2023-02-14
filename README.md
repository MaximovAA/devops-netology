# devops-netology
_____Задание 1_____

Просто смотрим коммит

amaksimov@git:~/devops-netology/terraform$ git show aefea  

commit aefead2207ef7e2aa5dc81a34aedf0cad4c32545  

Author: Alisdair McDiarmid <alisdair@users.noreply.github.com>  

Date:   Thu Jun 18 10:29:58 2020 -0400  

    Update CHANGELOG.md


_____Задание 2_____

Просто смотрим коммит

amaksimov@git:~/devops-netology/terraform$ git show 85024d3  

commit 85024d3100126de36331c6982bfaac02cdab9e76 (tag: v0.12.23)  

Author: tf-release-bot <terraform@hashicorp.com>  

Date:   Thu Mar 5 20:56:10 2020 +0000  


    v0.12.23

_____Задание 3_____

Нужный нам коммит был создан в резултате мерджа 2х коммитов 

amaksimov@git:~/devops-netology/terraform$ git show b8d720 --parents  

commit b8d720f8340221f2146e4e4870bf2ee0bc48f2d5 56cd7859e05c36c06b56d013b55a252d0bb7e158 9ea88f22fc6269854151c571162c5bcf958bee2b  

Merge: 56cd7859e0 9ea88f22fc  

Author: Chris Griggs <cgriggs@hashicorp.com>  

Date:   Tue Jan 21 17:45:48 2020 -0800  


    Merge pull request #23916 from hashicorp/cgriggs01-stable

    [Cherrypick] community links

    Merge: 56cd7859e0 9ea88f22fc


_____Задание 4_____

Смотрим лог, берем нужный диапазон

amaksimov@git:~/devops-netology/terraform$ git log v0.12.24 v0.12.23 --oneline  

>>>>>>>33ff1c03bb (tag: v0.12.24) v0.12.24
b14b74c493 [Website] vmc provider links
3f235065b9 Update CHANGELOG.md
6ae64e247b registry: Fix panic when server is unreachable
5c619ca1ba website: Remove links to the getting started guide's old location
06275647e2 Update CHANGELOG.md
d5f9411f51 command: Fix bug when using terraform login on Windows
4b6d06cc5d Update CHANGELOG.md
dd01a35078 Update CHANGELOG.md
225466bc3e Cleanup after v0.12.23 release
85024d3100 (tag: v0.12.23) v0.12.23<<<<<<<

_____Задание 5_____

```
Определяем где изменялась функция, находим коммит в котором она была создана

amaksimov@git:~/devops-netology/terraform$ git log -S'func providerSource' --oneline
5af1e6234a main: Honor explicit provider_installation CLI config when present
8c928e8358 main: Consult local directories as potential mirrors of providers
amaksimov@git:~/devops-netology/terraform$ git show 8c928e8358
commit 8c928e83589d90a031f811fae52a81be7153e82f
Author: Martin Atkins <mart@degeneration.co.uk>
Date:   Thu Apr 2 18:04:39 2020 -0700

    main: Consult local directories as potential mirrors of providers
```
_____Задание 6_____
```
Определяем файлы где упоминалась функция

git grep -n -e "globalPluginDirs"

Определяем в каких коммитах изменялась

amaksimov@git:~/devops-netology/terraform$ git log -L :globalPluginDirs:plugins.go   
commit 78b12205587fe839f10d946ea3fdc06719decb05
Author: Pam Selle <204372+pselle@users.noreply.github.com>
Date:   Mon Jan 13 16:50:05 2020 -0500

    Remove config.go and update things using its aliases

diff --git a/plugins.go b/plugins.go
--- a/plugins.go
+++ b/plugins.go
@@ -16,14 +18,14 @@
 func globalPluginDirs() []string {
        var ret []string
        // Look in ~/.terraform.d/plugins/ , or its equivalent on non-UNIX
-       dir, err := ConfigDir()
+       dir, err := cliconfig.ConfigDir()
        if err != nil {
                log.Printf("[ERROR] Error finding global config directory: %s", err)
        } else {
                machineDir := fmt.Sprintf("%s_%s", runtime.GOOS, runtime.GOARCH)
                ret = append(ret, filepath.Join(dir, "plugins"))
                ret = append(ret, filepath.Join(dir, "plugins", machineDir))
        }

        return ret
 }

commit 52dbf94834cb970b510f2fba853a5b49ad9b1a46
Author: James Bardin <j.bardin@gmail.com>
Date:   Wed Aug 9 17:46:49 2017 -0400

    keep .terraform.d/plugins for discovery

diff --git a/plugins.go b/plugins.go
--- a/plugins.go
+++ b/plugins.go
@@ -16,13 +16,14 @@
 func globalPluginDirs() []string {
        var ret []string
        // Look in ~/.terraform.d/plugins/ , or its equivalent on non-UNIX
        dir, err := ConfigDir()
        if err != nil {
                log.Printf("[ERROR] Error finding global config directory: %s", err)
        } else {
                machineDir := fmt.Sprintf("%s_%s", runtime.GOOS, runtime.GOARCH)
+               ret = append(ret, filepath.Join(dir, "plugins"))
                ret = append(ret, filepath.Join(dir, "plugins", machineDir))
        }

        return ret
 }

commit 41ab0aef7a0fe030e84018973a64135b11abcd70
Author: James Bardin <j.bardin@gmail.com>
Date:   Wed Aug 9 10:34:11 2017 -0400

    Add missing OS_ARCH dir to global plugin paths

    When the global directory was added, the discovery system still
    attempted to search for OS_ARCH subdirectories. It has since been
    changed only search explicit paths.

diff --git a/plugins.go b/plugins.go
--- a/plugins.go
+++ b/plugins.go
@@ -14,12 +16,13 @@
 func globalPluginDirs() []string {
        var ret []string
        // Look in ~/.terraform.d/plugins/ , or its equivalent on non-UNIX
        dir, err := ConfigDir()
        if err != nil {
                log.Printf("[ERROR] Error finding global config directory: %s", err)
        } else {
-               ret = append(ret, filepath.Join(dir, "plugins"))
+               machineDir := fmt.Sprintf("%s_%s", runtime.GOOS, runtime.GOARCH)
+               ret = append(ret, filepath.Join(dir, "plugins", machineDir))
        }

        return ret
 }

commit 66ebff90cdfaa6938f26f908c7ebad8d547fea17
Author: James Bardin <j.bardin@gmail.com>
Date:   Wed May 3 22:24:51 2017 -0400

    move some more plugin search path logic to command

    Make less to change when we remove the old search path

diff --git a/plugins.go b/plugins.go
--- a/plugins.go
+++ b/plugins.go
@@ -16,22 +14,12 @@
 func globalPluginDirs() []string {
        var ret []string
-
-       // Look in the same directory as the Terraform executable.
-       // If found, this replaces what we found in the config path.
-       exePath, err := osext.Executable()
-       if err != nil {
-               log.Printf("[ERROR] Error discovering exe directory: %s", err)
-       } else {
-               ret = append(ret, filepath.Dir(exePath))
-       }
-
        // Look in ~/.terraform.d/plugins/ , or its equivalent on non-UNIX
        dir, err := ConfigDir()
        if err != nil {
                log.Printf("[ERROR] Error finding global config directory: %s", err)
        } else {
                ret = append(ret, filepath.Join(dir, "plugins"))
        }

        return ret
 }

commit 8364383c359a6b738a436d1b7745ccdce178df47
Author: Martin Atkins <mart@degeneration.co.uk>
Date:   Thu Apr 13 18:05:58 2017 -0700

    Push plugin discovery down into command package

    Previously we did plugin discovery in the main package, but as we move
    towards versioned plugins we need more information available in order to
    resolve plugins, so we move this responsibility into the command package
    itself.

    For the moment this is just preserving the existing behavior as long as
    there are only internal and unversioned plugins present. This is the
    final state for provisioners in 0.10, since we don't want to support
    versioned provisioners yet. For providers this is just a checkpoint along
    the way, since further work is required to apply version constraints from
    configuration and support additional plugin search directories.

    The automatic plugin discovery behavior is not desirable for tests because
    we want to mock the plugins there, so we add a new backdoor for the tests
    to use to skip the plugin discovery and just provide their own mock
    implementations. Most of this diff is thus noisy rework of the tests to
    use this new mechanism.

diff --git a/plugins.go b/plugins.go
--- /dev/null
+++ b/plugins.go
@@ -0,0 +16,22 @@
+func globalPluginDirs() []string {
+       var ret []string
+
+       // Look in the same directory as the Terraform executable.
+       // If found, this replaces what we found in the config path.
+       exePath, err := osext.Executable()
+       if err != nil {
+               log.Printf("[ERROR] Error discovering exe directory: %s", err)
+       } else {
+               ret = append(ret, filepath.Dir(exePath))
+       }
+
+       // Look in ~/.terraform.d/plugins/ , or its equivalent on non-UNIX
+       dir, err := ConfigDir()
+       if err != nil {
+               log.Printf("[ERROR] Error finding global config directory: %s", err)
+       } else {
+               ret = append(ret, filepath.Join(dir, "plugins"))
+       }
+
+       return ret
+}
```

_____Задание 7_____
```
Определяем в каких коммитах добавляется выражение и просматриваем нужный
Author: Martin Atkins <mart@degeneration.co.uk>

amaksimov@git:~/devops-netology/terraform$ git log -S'synchronizedWriters' --oneline 
bdfea50cc8 remove unused
fd4f7eb0b9 remove prefixed io
5ac311e2a9 main: synchronize writes to VT100-faker on Windows
amaksimov@git:~/devops-netology/terraform$ git show 5ac311e2a9
commit 5ac311e2a91e381e2f52234668b49ba670aa0fe5
Author: Martin Atkins <mart@degeneration.co.uk>
Date:   Wed May 3 16:25:41 2017 -0700
```
