<!-- Old headings. Do not remove or links may break. -->

<a id="installing-binaries-from-cratesio-with-cargo-install"></a>

## تثبيت الملفات الثنائية (Installing Binaries) باستخدام `cargo install`

يسمح لك الأمر `cargo install` بتثبيت واستخدام الصناديق الثنائية (Binary Crates) محلياً. لا يهدف هذا الأمر إلى استبدال حزم النظام؛ بل يُقصد به أن يكون وسيلة مريحة لمطوري Rust لتثبيت الأدوات التي شاركها الآخرون على [crates.io](https://crates.io/)<!-- ignore -->. لاحظ أنه يمكنك فقط تثبيت الحزم التي تحتوي على أهداف ثنائية (Binary Targets). يعتبر *الهدف الثنائي* (Binary Target) هو البرنامج القابل للتشغيل الذي يتم إنشاؤه إذا كان الصندوق (Crate) يحتوي على ملف *src/main.rs* أو ملف آخر محدد كملف ثنائي، على عكس هدف المكتبة (Library Target) الذي لا يكون قابلاً للتشغيل بمفرده ولكنه مناسب للتضمين داخل برامج أخرى. عادةً ما تحتوي Crates على معلومات في ملف README حول ما إذا كان Crate عبارة عن مكتبة، أو يحتوي على Binary Target، أو كليهما.

يتم تخزين جميع الملفات الثنائية (Binaries) المثبتة باستخدام `cargo install` في مجلد *bin* الخاص بجذر التثبيت. إذا قمت بتثبيت Rust باستخدام *rustup.rs* ولم يكن لديك أي تكوينات مخصصة، فسيكون هذا الدليل هو *$HOME/.cargo/bin*. تأكد من أن هذا الدليل موجود في متغير البيئة `$PATH` الخاص بك لتتمكن من تشغيل البرامج التي قمت بتثبيتها باستخدام `cargo install`.

على سبيل المثال، ذكرنا في الفصل 12 أن هناك تنفيذاً بلغة Rust لأداة `grep` يسمى `ripgrep` للبحث في الملفات. لتثبيت `ripgrep` يمكننا تشغيل ما يلي:

<!-- manual-regeneration
cargo install something you don't have, copy relevant output below
-->

```console
$ cargo install ripgrep
    Updating crates.io index
  Downloaded ripgrep v14.1.1
  Downloaded 1 crate (213.6 KB) in 0.40s
  Installing ripgrep v14.1.1
--snip--
   Compiling grep v0.3.2
    Finished `release` profile [optimized + debuginfo] target(s) in 6.73s
  Installing ~/.cargo/bin/rg
   Installed package `ripgrep v14.1.1` (executable `rg`)
```

يُظهر السطر قبل الأخير من المخرجات موقع واسم Binary المثبت، والذي يكون في حالة `ripgrep` هو `rg`. طالما أن دليل التثبيت موجود في `$PATH` الخاص بك، كما ذكرنا سابقاً، يمكنك حينها تشغيل `rg --help` والبدء في استخدام أداة أسرع وأكثر ملاءمة لروح Rust للبحث في الملفات!
