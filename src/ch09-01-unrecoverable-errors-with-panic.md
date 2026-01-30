## الأخطاء غير القابلة للاسترداد باستخدام `panic!` (Unrecoverable Errors with `panic!`)

أحيانًا تحدث أشياء سيئة في الكود الخاص بك، ولا يوجد شيء يمكنك القيام به حيال ذلك. في هذه الحالات، تمتلك Rust ماكرو (macro) `panic!`. هناك طريقتان للتسبب في حدوث ذعر (panic) في الممارسة العملية: عن طريق اتخاذ إجراء يتسبب في ذعر الكود الخاص بنا (مثل الوصول إلى مصفوفة بعد نهايتها) أو عن طريق استدعاء ماكرو `panic!` صراحةً. في كلتا الحالتين، نتسبب في حدوث ذعر في برنامجنا. بشكل افتراضي، ستقوم حالات الذعر هذه بطباعة رسالة فشل، وفك المكدس (unwind)، وتنظيف المكدس (stack)، ثم الخروج. عبر متغير بيئة (environment variable)، يمكنك أيضًا جعل Rust تعرض مكدس الاستدعاءات (call stack) عند حدوث ذعر لتسهيل تتبع مصدر الذعر.

> ### فك المكدس أو الإجهاض استجابةً للذعر (Unwinding the Stack or Aborting in Response to a Panic)
>
> بشكل افتراضي، عندما يحدث ذعر، يبدأ البرنامج في *فك المكدس* (unwinding)، مما يعني أن Rust تعود إلى الوراء في المكدس وتنظف البيانات من كل دالة تصادفها. ومع ذلك، فإن العودة للخلف والتنظيف يتطلب الكثير من العمل. لذلك تسمح لك Rust باختيار البديل وهو *الإجهاض* (aborting) الفوري، والذي ينهي البرنامج دون تنظيف.
>
> ستحتاج الذاكرة التي كان البرنامج يستخدمها بعد ذلك إلى التنظيف بواسطة نظام التشغيل. إذا كنت بحاجة في مشروعك إلى جعل الملف الثنائي الناتج أصغر ما يمكن، فيمكنك التبديل من فك المكدس إلى الإجهاض عند حدوث ذعر عن طريق إضافة `panic = 'abort'` إلى أقسام `[profile]` المناسبة في ملف *Cargo.toml* الخاص بك. على سبيل المثال، إذا كنت تريد الإجهاض عند حدوث ذعر في وضع الإصدار (release mode)، فأضف هذا:
>
> ```toml
> [profile.release]
> panic = 'abort'
> ```

دعنا نحاول استدعاء `panic!` في برنامج بسيط:

<Listing file-name="src/main.rs">

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-01-panic/src/main.rs}}
```

</Listing>

عند تشغيل البرنامج، سترى شيئًا كهذا:

```console
{{#include ../listings/ch09-error-handling/no-listing-01-panic/output.txt}}
```

يتسبب استدعاء `panic!` في رسالة الخطأ الواردة في السطرين الأخيرين. يوضح السطر الأول رسالة الذعر الخاصة بنا والمكان في كود المصدر الخاص بنا حيث حدث الذعر: يشير _src/main.rs:2:5_ إلى أنه السطر الثاني، الحرف الخامس من ملف _src/main.rs_ الخاص بنا.

في هذه الحالة، السطر المشار إليه هو جزء من الكود الخاص بنا، وإذا ذهبنا إلى ذلك السطر، فسنرى استدعاء ماكرو `panic!`. في حالات أخرى، قد يكون استدعاء `panic!` في كود يستدعيه الكود الخاص بنا، وسيكون اسم الملف ورقم السطر الوارد في رسالة الخطأ هما كود شخص آخر حيث يتم استدعاء ماكرو `panic!` وليس سطر الكود الخاص بنا الذي أدى في النهاية إلى استدعاء `panic!`.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-a-panic-backtrace"></a>

يمكننا استخدام تتبع الخلفية (backtrace) للدوال التي جاء منها استدعاء `panic!` لمعرفة جزء الكود الخاص بنا الذي يسبب المشكلة. لفهم كيفية استخدام تتبع الخلفية لـ `panic!` دعنا ننظر إلى مثال آخر ونرى كيف يبدو الأمر عندما يأتي استدعاء `panic!` من مكتبة بسبب خطأ في الكود الخاص بنا بدلاً من استدعاء الكود الخاص بنا للماكرو مباشرةً. تحتوي القائمة 9-1 على بعض الكود الذي يحاول الوصول إلى فهرس (index) في متجه (vector) يتجاوز نطاق الفهارس الصالحة.

<Listing number="9-1" file-name="src/main.rs" caption="محاولة الوصول إلى عنصر يتجاوز نهاية المتجه، مما سيؤدي إلى استدعاء `panic!`">

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-01/src/main.rs}}
```

</Listing>

هنا، نحاول الوصول إلى العنصر رقم 100 في المتجه الخاص بنا (الموجود في الفهرس 99 لأن الفهرسة تبدأ من الصفر)، لكن المتجه يحتوي على ثلاثة عناصر فقط. في هذه الحالة، ستصاب Rust بالذعر. من المفترض أن يؤدي استخدام `[]` إلى إرجاع عنصر، ولكن إذا مررت فهرسًا غير صالح، فلا يوجد عنصر يمكن لـ Rust إرجاعه هنا ويكون صحيحًا.

في لغة C، تعد محاولة القراءة بعد نهاية هيكل البيانات سلوكًا غير محدد (undefined behavior). قد تحصل على أي شيء موجود في موقع الذاكرة الذي يتوافق مع ذلك العنصر في هيكل البيانات، على الرغم من أن الذاكرة لا تنتمي إلى ذلك الهيكل. يسمى هذا *قراءة زائدة للمخزن المؤقت* (buffer overread) ويمكن أن يؤدي إلى ثغرات أمنية إذا تمكن المهاجم من التلاعب بالفهرس بطريقة تمكنه من قراءة بيانات لا ينبغي السماح له بها والمخزنة بعد هيكل البيانات.

لحماية برنامجك من هذا النوع من الثغرات، إذا حاولت قراءة عنصر في فهرس غير موجود، فستوقف Rust التنفيذ وترفض الاستمرار. دعنا نجرب ذلك ونرى:

```console
{{#include ../listings/ch09-error-handling/listing-09-01/output.txt}}
```

يشير هذا الخطأ إلى السطر 4 من ملف _main.rs_ الخاص بنا حيث نحاول الوصول إلى الفهرس 99 للمتجه في `v`.

يخبرنا سطر `note:` أنه يمكننا تعيين متغير البيئة `RUST_BACKTRACE` للحصول على تتبع خلفية (backtrace) لما حدث بالضبط وتسبب في الخطأ. تتبع الخلفية هو قائمة بجميع الدوال التي تم استدعاؤها للوصول إلى هذه النقطة. تعمل تتبعات الخلفية في Rust كما تفعل في اللغات الأخرى: المفتاح لقراءة تتبع الخلفية هو البدء من الأعلى والقراءة حتى ترى الملفات التي كتبتها. هذه هي النقطة التي نشأت منها المشكلة. الأسطر الموجودة فوق تلك النقطة هي كود استدعاه الكود الخاص بك؛ الأسطر الموجودة أسفلها هي كود استدعى الكود الخاص بك. قد تتضمن هذه الأسطر السابقة واللاحقة كود Rust الأساسي (core)، أو كود المكتبة القياسية (standard library)، أو الصناديق (crates) التي تستخدمها. دعنا نحاول الحصول على تتبع خلفية عن طريق تعيين متغير البيئة `RUST_BACKTRACE` إلى أي قيمة باستثناء `0`. تعرض القائمة 9-2 مخرجات مشابهة لما ستراه.

<!-- manual-regeneration
cd listings/ch09-error-handling/listing-09-01
RUST_BACKTRACE=1 cargo run
copy the backtrace output below
check the backtrace number mentioned in the text below the listing
-->

<Listing number="9-2" caption="تتبع الخلفية الناتج عن استدعاء `panic!` المعروض عند تعيين متغير البيئة `RUST_BACKTRACE` ">

```console
$ RUST_BACKTRACE=1 cargo run
thread 'main' panicked at src/main.rs:4:6:
index out of bounds: the len is 3 but the index is 99
stack backtrace:
   0: rust_begin_unwind
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/std/src/panicking.rs:692:5
   1: core::panicking::panic_fmt
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/core/src/panicking.rs:75:14
   2: core::panicking::panic_bounds_check
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/core/src/panicking.rs:273:5
   3: <usize as core::slice::index::SliceIndex<[T]>>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/slice/index.rs:274:10
   4: core::slice::index::<impl core::ops::index::Index<I> for [T]>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/slice/index.rs:16:9
   5: <alloc::vec::Vec<T,A> as core::ops::index::Index<I>>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/alloc/src/vec/mod.rs:3361:9
   6: panic::main
             at ./src/main.rs:4:6
   7: core::ops::function::FnOnce::call_once
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/ops/function.rs:250:5
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
```

</Listing>

هذا الكثير من المخرجات! قد تختلف المخرجات الدقيقة التي تراها اعتمادًا على نظام التشغيل وإصدار Rust الخاص بك. من أجل الحصول على تتبعات خلفية بهذه المعلومات، يجب تمكين رموز التصحيح (debug symbols). يتم تمكين رموز التصحيح افتراضيًا عند استخدام `cargo build` أو `cargo run` بدون علم `--release` كما فعلنا هنا.

في المخرجات في القائمة 9-2، يشير السطر 6 من تتبع الخلفية إلى السطر في مشروعنا الذي يسبب المشكلة: السطر 4 من _src/main.rs_. إذا كنا لا نريد أن يصاب برنامجنا بالذعر، فيجب أن نبدأ تحقيقنا في الموقع الذي يشير إليه السطر الأول الذي يذكر ملفًا كتبناه. في القائمة 9-1، حيث كتبنا عمدًا كودًا من شأنه أن يسبب ذعرًا، فإن طريقة إصلاح الذعر هي عدم طلب عنصر يتجاوز نطاق فهارس المتجه. عندما يصاب الكود الخاص بك بالذعر في المستقبل، ستحتاج إلى معرفة الإجراء الذي يتخذه الكود مع أي قيم للتسبب في الذعر وما يجب أن يفعله الكود بدلاً من ذلك.

سنعود إلى `panic!` ومتى يجب ولا يجب استخدام `panic!` للتعامل مع ظروف الخطأ في قسم ["استخدام `panic!` أو عدم استخدامه"][to-panic-or-not-to-panic]<!-- ignore --> لاحقًا في هذا الفصل. بعد ذلك، سننظر في كيفية الاسترداد من خطأ باستخدام `Result`.

[to-panic-or-not-to-panic]: ch09-03-to-panic-or-not-to-panic.html#to-panic-or-not-to-panic
