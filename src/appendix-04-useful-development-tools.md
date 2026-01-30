## الملحق د: أدوات التطوير المفيدة (Useful Development Tools)

في هذا الملحق، نتحدث عن بعض أدوات التطوير المفيدة التي يوفرها مشروع Rust. سننظر في التنسيق التلقائي (Automatic Formatting)، والطرق السريعة لتطبيق إصلاحات تحذيرات المترجم (compiler warnings)، وأداة التحقق من الكود (linter)، والتكامل مع بيئات التطوير المتكاملة (IDEs).

### التنسيق التلقائي باستخدام `rustfmt`

أداة تنسيق الكود (rustfmt) تعيد تنسيق الكود الخاص بك وفقًا لنمط الكود (code style) الخاص بالمجتمع. تستخدم العديد من المشاريع التعاونية `rustfmt` لمنع الجدال حول النمط الذي يجب استخدامه عند كتابة Rust: يقوم الجميع بتنسيق الكود الخاص بهم باستخدام الأداة.

تتضمن تثبيتات Rust أداة `rustfmt` بشكل افتراضي، لذلك يجب أن تكون لديك بالفعل البرنامجان `rustfmt` وأمر تنسيق الكود (cargo-fmt) على نظامك. هذان الأمران يشبهان مترجم لغة رست (rustc) ومدير الحزم (cargo) من حيث أن `rustfmt` يسمح بتحكم أدق، بينما `cargo-fmt` يفهم اتفاقيات المشروع الذي يستخدم `cargo`. لتنسيق أي مشروع `cargo`، أدخل ما يلي:

```console
$ cargo fmt
```

تشغيل هذا الأمر يعيد تنسيق كل كود Rust في الحزمة (crate) الحالية. يجب أن يغير هذا فقط نمط الكود، وليس دلالات الكود (code semantics). لمزيد من المعلومات حول `rustfmt`، راجع [وثائقه][rustfmt].

### إصلاح الكود الخاص بك باستخدام `rustfix`

أداة إصلاح الكود (rustfix) مُضمنة مع تثبيتات Rust ويمكنها تلقائيًا إصلاح تحذيرات المترجم (compiler warnings) التي لديها طريقة واضحة لتصحيح المشكلة، والتي من المحتمل أن تكون ما تريده. ربما تكون قد رأيت `compiler warnings` من قبل. على سبيل المثال، لننظر إلى هذا الكود:

<span class="filename">اسم الملف: src/main.rs</span>

```rust
fn main() {
    let mut x = 42;
    println!("{x}");
}
```

هنا، نقوم بتعريف المتغير `x` على أنه قابل للتغيير (mutable)، لكننا لا نغيره أبدًا. تحذرنا Rust من ذلك:

```console
$ cargo build
   Compiling myprogram v0.1.0 (file:///projects/myprogram)
warning: variable does not need to be mutable
 --> src/main.rs:2:9
  |
52 |     let mut x = 0;
  |         ----^
  |         |
  |         help: remove this `mut`
  |
  = note: `#[warn(unused_mut)]` on by default
```

يقترح التحذير أن نزيل الكلمة المفتاحية `mut`. يمكننا تطبيق هذا الاقتراح تلقائيًا باستخدام أداة `rustfix` عن طريق تشغيل الأمر `cargo fix`:

```console
$ cargo fix
    Checking myprogram v0.1.0 (file:///projects/myprogram)
      Fixing src/main.rs (1 fix)
    Finished dev [unoptimized + debuginfo] target(s) in 0.59s
```

عندما ننظر إلى _src/main.rs_ مرة أخرى، سنرى أن `cargo fix` قد غير الكود:

<span class="filename">اسم الملف: src/main.rs</span>

```rust
fn main() {
    let x = 42;
    println!("{x}");
}
```

المتغير `x` أصبح الآن غير قابل للتغيير (immutable)، ولم يعد التحذير يظهر.

يمكنك أيضًا استخدام الأمر `cargo fix` لنقل الكود الخاص بك بين إصدارات لغة رست (Rust editions) المختلفة. يتم تناول `Rust editions` في [الملحق هـ][editions]<!-- ignore -->.

### المزيد من قواعد التحقق (Lints) باستخدام Clippy

أداة Clippy (Clippy) هي مجموعة من `Lints` لتحليل الكود الخاص بك حتى تتمكن من اكتشاف الأخطاء الشائعة وتحسين كود Rust الخاص بك. `Clippy` مُضمنة مع تثبيتات Rust القياسية.

لتشغيل `Lints` الخاصة بـ `Clippy` على أي مشروع `cargo`، أدخل ما يلي:

```console
$ cargo clippy
```

على سبيل المثال، لنفترض أنك كتبت برنامجًا يستخدم تقريبًا لثابت رياضي، مثل باي (pi)، كما يفعل هذا البرنامج:

<Listing file-name="src/main.rs">

```rust
fn main() {
    let x = 3.1415;
    let r = 8.0;
    println!("the area of the circle is {}", x * r * r);
}
```

</Listing>

تشغيل `cargo clippy` على هذا المشروع ينتج عنه هذا الخطأ:

```text
error: approximate value of `f{32, 64}::consts::PI` found
 --> src/main.rs:2:13
  |
2 |     let x = 3.1415;
  |             ^^^^^^
  |
  = note: `#[deny(clippy::approx_constant)]` on by default
  = help: consider using the constant directly
  = help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#approx_constant
```

يخبرك هذا الخطأ أن Rust لديها بالفعل ثابت `PI` أكثر دقة مُعرَّف، وأن برنامجك سيكون أكثر صحة إذا استخدمت الثابت بدلاً من ذلك. ستقوم بعد ذلك بتغيير الكود الخاص بك لاستخدام الثابت `PI`.

الكود التالي لا ينتج عنه أي أخطاء أو تحذيرات من `Clippy`:

<Listing file-name="src/main.rs">

```rust
fn main() {
    let x = std::f64::consts::PI;
    let r = 8.0;
    println!("the area of the circle is {}", x * r * r);
}
```

</Listing>

لمزيد من المعلومات حول `Clippy`، راجع [وثائقه][clippy].

### تكامل بيئة التطوير المتكاملة (IDE Integration) باستخدام `rust-analyzer`

للمساعدة في `IDE Integration`، يوصي مجتمع Rust باستخدام محلل لغة رست (rust-analyzer). هذه الأداة هي مجموعة من الأدوات المساعدة التي تركز على المترجم (compiler-centric utilities) وتتحدث بروتوكول خادم اللغة (Language Server Protocol)، وهو مواصفات لـ `IDEs` ولغات البرمجة للتواصل مع بعضها البعض. يمكن لعملاء مختلفين استخدام `rust-analyzer`، مثل [المكون الإضافي (plug-in) لـ `rust-analyzer` لـ Visual Studio Code][vscode].

قم بزيارة [الصفحة الرئيسية][rust-analyzer] لمشروع `rust-analyzer` للحصول على تعليمات التثبيت، ثم قم بتثبيت دعم خادم اللغة (language server support) في `IDE` الخاص بك. سيكتسب `IDE` الخاص بك قدرات مثل الإكمال التلقائي (autocompletion)، والانتقال إلى التعريف (jump to definition)، والأخطاء المضمنة (inline errors).

[rustfmt]: https://github.com/rust-lang/rustfmt
[editions]: appendix-05-editions.md
[clippy]: https://github.com/rust-lang/rust-clippy
[rust-analyzer]: https://rust-analyzer.github.io
[lsp]: http://langserver.org/
[vscode]: https://marketplace.visualstudio.com/items?itemName=rust-lang.rust-analyzer
