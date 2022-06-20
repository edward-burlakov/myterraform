# Домашнее задание к занятию «2.4. Инструменты Git»


# 1.Найдите полный хеш и комментарий коммита, хеш которого начинается на aefea.
>git log aefea -n 1

    commit aefead2207ef7e2aa5dc81a34aedf0cad4c32545
    Author: Alisdair McDiarmid <alisdair@users.noreply.github.com>
    Date:   Thu Jun 18 10:29:58 2020 -0400

        Update CHANGELOG.md

# 2. Какому тегу соответствует коммит 85024d3? 
>git log 85024d3 --oneline -n 1

    85024d310 (tag: v0.12.23) v0.12.23
# Ответ: v0.12.23

# 3. Сколько родителей у коммита b8d720? Напишите их хеши.
>git log b8d720 --oneline --graph

    9ea88f22f add/update community provider listings
    56cd7859e Merge pull request #23857 from hashicorp/cgriggs01-stable

# 4. Перечислите хеши и комментарии всех коммитов которые были сделаны между тегами v0.12.23 и v0.12.24.

# Вариант 1 

>git log v0.12.23..v0.12.24 --oneline

    33ff1c03b (tag: v0.12.24) v0.12.24
    b14b74c49 [Website] vmc provider links
    3f235065b Update CHANGELOG.md
    6ae64e247 registry: Fix panic when server is unreachable
    5c619ca1b website: Remove links to the getting started guide's old location
    06275647e Update CHANGELOG.md
    d5f9411f5 command: Fix bug when using terraform login on Windows
    4b6d06cc5 Update CHANGELOG.md
    dd01a3507 Update CHANGELOG.md
    225466bc3 Cleanup after v0.12.23 release



# Вариант 2

>git show v0.12.23..v0.12.24 --oneline --no-patch

    33ff1c03b (tag: v0.12.24) v0.12.24
    b14b74c49 [Website] vmc provider links
    3f235065b Update CHANGELOG.md
    6ae64e247 registry: Fix panic when server is unreachable
    5c619ca1b website: Remove links to the getting started guide's old location
    06275647e Update CHANGELOG.md
    d5f9411f5 command: Fix bug when using terraform login on Windows
    4b6d06cc5 Update CHANGELOG.md
    dd01a3507 Update CHANGELOG.md
    225466bc3 Cleanup after v0.12.23 release

# Вариант 3 
>git show --format=full v0.12.23

    commit 85024d3100126de36331c6982bfaac02cdab9e76 (tag: v0.12.23)

>git show --format=full v0.12.24

    commit 33ff1c03bb960b332be3af2e333462dde88b279e (tag: v0.12.24)
 
> git log 33ff1c03b  --not 85024d310 --oneline

    33ff1c03b (tag: v0.12.24) v0.12.24
    b14b74c49 [Website] vmc provider links
    3f235065b Update CHANGELOG.md
    6ae64e247 registry: Fix panic when server is unreachable
    5c619ca1b website: Remove links to the getting started guide's old location
    06275647e Update CHANGELOG.md
    d5f9411f5 command: Fix bug when using terraform login on Windows
    4b6d06cc5 Update CHANGELOG.md
    dd01a3507 Update CHANGELOG.md
    225466bc3 Cleanup after v0.12.23 release

# 5. Найдите коммит, в котором была создана функция func providerSource, ее определение в коде выглядит так func providerSource(...) (вместо троеточего перечислены аргументы).

# Вариант 1 (кратко):
>git log -S"func providerSource(" --oneline

   8c928e835 main: Consult local directories as potential mirrors of providers


# Вариант 2:

# Ищем исходники, в которых используется функция:
>git grep -n "func providerSource("

  provider_source.go:23:func providerSource(configs []*cliconfig.ProviderInstallation, services *disco.Disco) (getproviders.Source, tfdiags.Diagnostics) { 

# Находим коммит, где функция была добавлена:
>git log -L :providerSource:provider_source.go --oneline
# ==========================================
    8c928e835 main: Consult local directories as potential mirrors of providers

    diff --git a/provider_source.go b/provider_source.go
    --- /dev/null
    +++ b/provider_source.go
    @@ -0,0 +19,5 @@
    +func providerSource(services *disco.Disco) getproviders.Source {
    +   // We're not yet using the CLI config here because we've not implemented
    +      // yet the new configuration constructs to customize provider search
    +       // locations. That'll come later.
    +       // For now, we have a fixed set of search directories:
# ==========================================


# 6. Найдите все коммиты, в которых была изменена функция globalPluginDirs.

>git grep  "func globalPluginDirs("
# Ищем исходник, в котором используется функция:
plugins.go:func globalPluginDirs() []string {

>git log -L :globalPluginDirs:plugins.go --oneline
#  Коммиты, в которых редактировалась функция globalPluginDirs
#  78b122055
#  52dbf9483 
#  41ab0aef7
#  66ebff90c
#  8364383c3 (Описание функции)
# =========================================
78b122055 Remove config.go and update things using its aliases

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
#
52dbf9483 keep .terraform.d/plugins for discovery

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
#
41ab0aef7 Add missing OS_ARCH dir to global plugin paths

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
#
66ebff90c move some more plugin search path logic to command

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
#
8364383c3 Push plugin discovery down into command package

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
#=======================================

# 7. Кто автор функции synchronizedWriters?

>git log -S"synchronizedWriters"  --oneline
    bdfea50cc remove unused
    fd4f7eb0b remove prefixed io
    5ac311e2a main: synchronize writes to VT100-faker on Windows
>git log -S"synchronizedWriters" 5ac311e2a
    commit 5ac311e2a91e381e2f52234668b49ba670aa0fe5
    Author: Martin Atkins <mart@degeneration.co.uk>
    Date:   Wed May 3 16:25:41 2017 -0700

# Ответ:  Martin Atkins
