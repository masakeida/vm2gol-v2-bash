If you would like to run it on FreeBSD, ...


```
% env LANG=C bash --version | grep bash
GNU bash, version 5.1.12(0)-release (amd64-portbld-freebsd12.2)
```

```
% git clone --recursive https://github.com/masakeida/vm2gol-v2-bash.git
% cd vm2gol-v2-bash
% ./test.sh all
```

as a patch file.
```
diff -uprN vm2gol-v2-bash_orig/lib/utils.sh vm2gol-v2-bash/lib/utils.sh
--- vm2gol-v2-bash_orig/lib/utils.sh	2022-02-01 16:43:47.981015000 +0000
+++ vm2gol-v2-bash/lib/utils.sh	2022-02-01 16:44:41.911766000 +0000
@@ -1,5 +1,5 @@
 _to_hex() {
-  od --address-radix=n --format=x1 --output-duplicates \
+  od -A n -t x1 -v \
     | tr " " "\n" \
     | grep -v '^$'
 }
diff -uprN vm2gol-v2-bash_orig/test.sh vm2gol-v2-bash/test.sh
--- vm2gol-v2-bash_orig/test.sh	2022-02-01 16:43:48.009878000 +0000
+++ vm2gol-v2-bash/test.sh	2022-02-01 16:44:12.412555000 +0000
@@ -1,4 +1,4 @@
-#!/bin/bash
+#!/usr/bin/env bash
 
 set -o nounset
 
@@ -22,7 +22,7 @@ MAX_ID_COMPILE=27
 
 ERRS=""
 
-BASH_CMD=/bin/bash
+BASH_CMD="/usr/bin/env bash"
 
 run_test_utils() {
   $BASH_CMD test/test_utils.sh
```

### FreeBSD で動かすために

sonota88 さんの vm2gol-v2-bash が面白そうだったので、FreeBSD で動かしてみたいと思い、修正しました。基本的に変更点は2つ。`bash` の場所と `od` のオプションの書き方です。

FreeBSD の `bash` は `/usr/local/bin/bash` にあるので、絶対 PATH の代わりに、`/usr/bin/env bash` を使うと、Linux でも FreeBSD でも動きます。

`test.sh` の1行目と25行目をそれぞれ以下のように変更しました。
```
-#!/bin/bash
+#!/usr/bin/env bash
```
```
-BASH_CMD=/bin/bash
+BASH_CMD="/usr/bin/env bash"
```


FreeBSD の `od` はロングオプションが使えません。ただショートオプションで同じ動作をするものがあるので、ショートオプションに変更しました。GNU `od` でも同じ動きをするので、Linux でも FreeBSD でも動きます。

`lib/utils.sh` の2行目を以下のように変更しました。
```
-  od --address-radix=n --format=x1 --output-duplicates \
+  od -A n -t x1 -v \
```

`lexer.sh`, `parser.sh`, `codegen.sh` は shebang が書かれてないので、`tcsh` などからは、
```
% cat gol.vg.txt | bash lexer.sh | bash parser.sh | bash codegen.sh
```
のように使うといいと思います。

以上の変更で、`./test.sh all` が Ubuntu と FreeBSD の両方で通りました。
