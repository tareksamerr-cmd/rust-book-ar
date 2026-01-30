## الماكرو (Macros)

لقد استخدمنا macros مثل `println!` في جميع أنحاء هذا الكتاب، لكننا لم نستكشف بالكامل ما هو الـ macro وكيف يعمل. يشير مصطلح *macro* إلى مجموعة من الميزات في Rust - الـ macros التصريحية (declarative macros) باستخدام `macro_rules!` وثلاثة أنواع من الـ macros الإجرائية (procedural macros):

- الـ custom `#[derive]` macros التي تحدد الكود المضاف باستخدام السمة (attribute) `derive` المستخدمة على الـ structs والـ enums.
- الـ attribute-like macros التي تحدد attributes مخصصة قابلة للاستخدام على أي عنصر.
- الـ function-like macros التي تبدو وكأنها استدعاءات دالة ولكنها تعمل على الرموز (tokens) المحددة كوسيطة لها.

سنتحدث عن كل من هذه الأنواع بالتتابع، ولكن أولاً، دعنا ننظر إلى سبب حاجتنا إلى الـ macros بينما لدينا بالفعل دوال (functions).

### الفرق بين الـ Macros والدوال

بشكل أساسي، الـ macros هي طريقة لكتابة كود يكتب كودًا آخر، وهو ما يُعرف باسم *البرمجة الوصفية* (metaprogramming). في الملحق ج، نناقش الـ attribute `derive`، الذي يولد تطبيقًا (implementation) لسمات (traits) مختلفة لك. لقد استخدمنا أيضًا الـ macros `println!` و `vec!` في جميع أنحاء الكتاب. كل هذه الـ macros *تتوسع* (expand) لإنتاج كود أكثر من الكود الذي كتبته يدويًا.

الـ metaprogramming مفيدة لتقليل كمية الكود الذي يتعين عليك كتابته وصيانته، وهو أيضًا أحد أدوار الـ functions. ومع ذلك، تتمتع الـ macros ببعض الصلاحيات الإضافية التي لا تتمتع بها الـ functions.

يجب أن يحدد توقيع الـ function عدد ونوع المعلمات (parameters) التي تحتوي عليها الـ function. من ناحية أخرى، يمكن أن تأخذ الـ macros عددًا متغيرًا من الـ parameters: يمكننا استدعاء `println!("hello")` بوسيطة واحدة أو `println!("hello {}", name)` بوسيطتين. أيضًا، يتم توسيع الـ macros قبل أن يفسر المترجم (compiler) معنى الكود، لذلك يمكن للـ macro، على سبيل المثال، تطبيق trait على نوع معين. لا يمكن للـ function ذلك، لأنه يتم استدعاؤها في وقت التشغيل (runtime) ويجب تطبيق الـ trait في وقت التجميع (compile time).

الجانب السلبي لتطبيق macro بدلاً من function هو أن تعريفات الـ macro أكثر تعقيدًا من تعريفات الـ function لأنك تكتب كود Rust يكتب كود Rust. بسبب هذا التوسط (indirection)، تكون تعريفات الـ macro بشكل عام أكثر صعوبة في القراءة والفهم والصيانة من تعريفات الـ function.

هناك فرق مهم آخر بين الـ macros والـ functions وهو أنه يجب عليك تعريف الـ macros أو إحضارها إلى النطاق (scope) *قبل* استدعائها في ملف، على عكس الـ functions التي يمكنك تعريفها في أي مكان واستدعائها في أي مكان.

<!-- Old headings. Do not remove or links may break. -->

<a id="declarative-macros-with-macro_rules-for-general-metaprogramming"></a>

### الـ Declarative Macros للـ Metaprogramming العام

الشكل الأكثر استخدامًا للـ macros في Rust هو *الـ declarative macro*. يشار إليها أحيانًا باسم "macros by example" أو "`macro_rules!` macros" أو ببساطة "macros". في جوهرها، تسمح لك الـ declarative macros بكتابة شيء مشابه لتعبير `match` في Rust. كما نوقش في الفصل 6، تعبيرات `match` هي هياكل تحكم تأخذ تعبيرًا، وتقارن القيمة الناتجة للتعبير بالأنماط (patterns)، ثم تقوم بتشغيل الكود المرتبط بالـ pattern المطابق. تقارن الـ macros أيضًا قيمة بالـ patterns المرتبطة بكود معين: في هذا الموقف، القيمة هي كود مصدر Rust الحرفي الذي تم تمريره إلى الـ macro؛ تتم مقارنة الـ patterns بهيكل كود المصدر هذا؛ والكود المرتبط بكل pattern، عند مطابقته، يحل محل الكود الذي تم تمريره إلى الـ macro. يحدث كل هذا أثناء الـ compilation.

لتعريف macro، تستخدم البنية `macro_rules!`. دعنا نستكشف كيفية استخدام `macro_rules!` من خلال النظر في كيفية تعريف الـ macro `vec!`. غطى الفصل 8 كيف يمكننا استخدام الـ macro `vec!` لإنشاء متجه (vector) جديد بقيم معينة. على سبيل المثال، ينشئ الـ macro التالي vector جديدًا يحتوي على ثلاثة أعداد صحيحة:

```rust
let v: Vec<u32> = vec![1, 2, 3];
```

يمكننا أيضًا استخدام الـ macro `vec!` لإنشاء vector من عددين صحيحين أو vector من خمس شرائح string (string slices). لن نتمكن من استخدام function للقيام بنفس الشيء لأننا لن نعرف عدد أو نوع القيم مقدمًا.

تُظهر القائمة 20-35 تعريفًا مبسطًا قليلاً للـ macro `vec!`.

<Listing number="20-35" file-name="src/lib.rs" caption="نسخة مبسطة من تعريف الـ macro `vec!`">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-35/src/lib.rs}}
```

</Listing>

> ملاحظة: يتضمن التعريف الفعلي للـ macro `vec!` في المكتبة القياسية كودًا لتخصيص الكمية الصحيحة من الذاكرة مقدمًا. هذا الكود هو تحسين (optimization) لا ندرجه هنا، لجعل المثال أبسط.

يشير التعليق التوضيحي `#[macro_export]` إلى أنه يجب إتاحة هذا الـ macro كلما تم إحضار الصندوق (crate) الذي تم تعريف الـ macro فيه إلى الـ scope. بدون هذا التعليق التوضيحي، لا يمكن إحضار الـ macro إلى الـ scope.

نبدأ بعد ذلك تعريف الـ macro بـ `macro_rules!` واسم الـ macro الذي نقوم بتعريفه *بدون* علامة التعجب. يتبع الاسم، في هذه الحالة `vec`، بأقواس متعرجة تشير إلى نص تعريف الـ macro.

الهيكل في نص `vec!` مشابه لهيكل تعبير `match`. لدينا هنا ذراع (arm) واحد مع الـ pattern `( $( $x:expr ),* )`، متبوعًا بـ `=>` وكتلة الكود المرتبطة بهذا الـ pattern. إذا تطابق الـ pattern، فسيتم إصدار كتلة الكود المرتبطة. نظرًا لأن هذا هو الـ pattern الوحيد في هذا الـ macro، فهناك طريقة واحدة صالحة فقط للمطابقة؛ سيؤدي أي pattern آخر إلى حدوث خطأ. سيكون للـ macros الأكثر تعقيدًا أكثر من ذراع واحد.

يختلف بناء جملة الـ pattern الصالح في تعريفات الـ macro عن بناء جملة الـ pattern الذي تم تناوله في الفصل 19 لأن الـ macro patterns تتم مطابقتها مع هيكل كود Rust بدلاً من القيم. دعنا نطلع على ما تعنيه أجزاء الـ pattern في القائمة 20-29؛ للحصول على بناء جملة الـ macro pattern الكامل، راجع [مرجع Rust][ref].

أولاً، نستخدم مجموعة من الأقواس لاحتواء الـ pattern بالكامل. نستخدم علامة الدولار (`$`) للإعلان عن متغير في نظام الـ macro سيحتوي على كود Rust الذي يطابق الـ pattern. تجعل علامة الدولار من الواضح أن هذا متغير macro بدلاً من متغير Rust عادي. بعد ذلك تأتي مجموعة من الأقواس التي تلتقط القيم التي تطابق الـ pattern داخل الأقواس لاستخدامها في كود الاستبدال. داخل `$()` يوجد `$x:expr`، الذي يطابق أي تعبير Rust ويعطي التعبير الاسم `$x`.

تشير الفاصلة التي تلي `$()` إلى أنه يجب أن تظهر فاصلة حرفية فاصلة بين كل مثيل للكود الذي يطابق الكود في `$()`. يحدد الرمز `*` أن الـ pattern يطابق صفرًا أو أكثر مما يسبق الرمز `*`.

عندما نستدعي هذا الـ macro بـ `vec![1, 2, 3];`، يطابق الـ pattern `$x` ثلاث مرات مع التعبيرات الثلاثة `1` و `2` و `3`.

الآن دعنا ننظر إلى الـ pattern في نص الكود المرتبط بهذا الـ arm: يتم إنشاء `temp_vec.push()` داخل `$()*` لكل جزء يطابق `$()` في الـ pattern صفرًا أو أكثر من المرات اعتمادًا على عدد المرات التي يطابق فيها الـ pattern. يتم استبدال `$x` بكل تعبير مطابق. عندما نستدعي هذا الـ macro بـ `vec![1, 2, 3];`، سيكون الكود الذي تم إنشاؤه والذي يحل محل استدعاء الـ macro هذا هو التالي:

```rust,ignore
{
    let mut temp_vec = Vec::new();
    temp_vec.push(1);
    temp_vec.push(2);
    temp_vec.push(3);
    temp_vec
}
```

لقد قمنا بتعريف macro يمكنه أخذ أي عدد من الوسائط من أي نوع ويمكنه إنشاء كود لإنشاء vector يحتوي على العناصر المحددة.

لمعرفة المزيد حول كيفية كتابة الـ macros، راجع الوثائق عبر الإنترنت أو الموارد الأخرى، مثل ["The Little Book of Rust Macros"][tlborm] الذي بدأه Daniel Keep وواصله Lukas Wirth.

### الـ Procedural Macros لإنشاء الكود من الـ Attributes

الشكل الثاني من الـ macros هو الـ procedural macro، الذي يعمل أشبه بـ function (وهو نوع من الإجراءات). تقبل الـ *procedural macros* بعض الكود كمدخل، وتعمل على هذا الكود، وتنتج بعض الكود كمخرج بدلاً من المطابقة مع الـ patterns واستبدال الكود بكود آخر كما تفعل الـ declarative macros. الأنواع الثلاثة من الـ procedural macros هي `derive` المخصص، والـ attribute-like، والـ function-like، وجميعها تعمل بطريقة مماثلة.

عند إنشاء الـ procedural macros، يجب أن توجد التعريفات في صندوقها الخاص بنوع صندوق خاص. هذا لأسباب تقنية معقدة نأمل في التخلص منها في المستقبل. في القائمة 20-36، نوضح كيفية تعريف procedural macro، حيث `some_attribute` هو عنصر نائب لاستخدام نوع macro معين.

<Listing number="20-36" file-name="src/lib.rs" caption="مثال على تعريف procedural macro">

```rust,ignore
use proc_macro::TokenStream;

#[some_attribute]
pub fn some_name(input: TokenStream) -> TokenStream {
}
```

</Listing>

تأخذ الـ function التي تحدد procedural macro `TokenStream` كمدخل وتنتج `TokenStream` كمخرج. يتم تعريف النوع `TokenStream` بواسطة الـ crate `proc_macro` المضمن مع Rust ويمثل تسلسلًا من الـ tokens. هذا هو جوهر الـ macro: الكود المصدري الذي يعمل عليه الـ macro يشكل الـ input `TokenStream`، والكود الذي ينتجه الـ macro هو الـ output `TokenStream`. تحتوي الـ function أيضًا على attribute مرفق بها يحدد نوع الـ procedural macro الذي نقوم بإنشائه. يمكن أن يكون لدينا أنواع متعددة من الـ procedural macros في نفس الـ crate.

دعنا ننظر إلى الأنواع المختلفة من الـ procedural macros. سنبدأ بـ custom `derive` macro ثم نشرح الاختلافات الصغيرة التي تجعل الأشكال الأخرى مختلفة.

<!-- Old headings. Do not remove or links may break. -->

<a id="how-to-write-a-custom-derive-macro"></a>

### الـ Custom `derive` Macros

دعنا ننشئ crate يسمى `hello_macro` يحدد trait يسمى `HelloMacro` مع function واحدة مرتبطة تسمى `hello_macro`. بدلاً من جعل مستخدمينا يطبقون الـ trait `HelloMacro` لكل نوع من أنواعهم، سنوفر procedural macro بحيث يمكن للمستخدمين إضافة تعليق توضيحي لنوعهم بـ `#[derive(HelloMacro)]` للحصول على تطبيق افتراضي لـ function `hello_macro`. سيطبع التطبيق الافتراضي `Hello, Macro! My name is TypeName!` حيث `TypeName` هو اسم النوع الذي تم تعريف هذا الـ trait عليه. بعبارة أخرى، سنكتب crate يمكّن مبرمجًا آخر من كتابة كود مثل القائمة 20-37 باستخدام الـ crate الخاص بنا.

<Listing number="20-37" file-name="src/main.rs" caption="الكود الذي سيتمكن مستخدم الـ crate الخاص بنا من كتابته عند استخدام الـ procedural macro الخاص بنا">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-37/src/main.rs}}
```

</Listing>

سيقوم هذا الكود بطباعة `Hello, Macro! My name is Pancakes!` عندما ننتهي. الخطوة الأولى هي إنشاء library crate جديد، مثل هذا:

```console
$ cargo new hello_macro --lib
```

<Listing file-name="src/lib.rs" number="20-38" caption="trait بسيط سنستخدمه مع الـ macro `derive`">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-38/hello_macro/src/lib.rs}}
```

</Listing>

لدينا trait و function خاص به. في هذه المرحلة، يمكن لمستخدم الـ crate الخاص بنا تطبيق الـ trait لتحقيق الوظيفة المطلوبة، كما في القائمة 20-39.

<Listing number="20-39" file-name="src/main.rs" caption="كيف سيبدو الأمر إذا كتب المستخدمون تطبيقًا يدويًا لـ trait `HelloMacro`">

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-39/pancakes/src/main.rs}}
```

</Listing>

ومع ذلك، سيحتاجون إلى كتابة كتلة التطبيق لكل نوع يريدون استخدامه مع `hello_macro`؛ نريد أن نوفر عليهم القيام بهذا العمل.

بالإضافة إلى ذلك، لا يمكننا حتى الآن توفير function `hello_macro` بتطبيق افتراضي سيطبع اسم النوع الذي تم تطبيق الـ trait عليه: لا تمتلك Rust إمكانيات الانعكاس (reflection)، لذلك لا يمكنها البحث عن اسم النوع في الـ runtime. نحتاج إلى macro لإنشاء كود في الـ compile time.

الخطوة التالية هي تعريف الـ procedural macro. في وقت كتابة هذا التقرير، يجب أن تكون الـ procedural macros في الـ crate الخاص بها. في النهاية، قد يتم رفع هذا القيد. الاصطلاح لهيكلة الـ crates والـ macro crates هو كما يلي: بالنسبة لـ crate يسمى `foo`، يسمى الـ custom `derive` procedural macro crate بـ `foo_derive`. دعنا نبدأ crate جديدًا يسمى `hello_macro_derive` داخل مشروع `hello_macro` الخاص بنا:

```console
$ cargo new hello_macro_derive --lib
```

الـ crates الخاصان بنا مرتبطان ارتباطًا وثيقًا، لذلك نقوم بإنشاء الـ procedural macro crate داخل دليل الـ crate `hello_macro` الخاص بنا. إذا قمنا بتغيير تعريف الـ trait في `hello_macro`، فسيتعين علينا تغيير تطبيق الـ procedural macro في `hello_macro_derive` أيضًا. سيحتاج الـ crates إلى النشر بشكل منفصل، وسيحتاج المبرمجون الذين يستخدمون هذه الـ crates إلى إضافتها كـ dependencies وإحضار كليهما إلى الـ scope. يمكننا بدلاً من ذلك أن نجعل الـ crate `hello_macro` يستخدم `hello_macro_derive` كـ dependency ويعيد تصدير كود الـ procedural macro. ومع ذلك، فإن الطريقة التي قمنا بها بهيكلة المشروع تجعل من الممكن للمبرمجين استخدام `hello_macro` حتى لو لم يرغبوا في وظيفة `derive`.

نحتاج إلى الإعلان عن الـ crate `hello_macro_derive` كـ procedural macro crate. سنحتاج أيضًا إلى وظيفة من الـ crates `syn` و `quote`، كما سترى بعد قليل، لذلك نحتاج إلى إضافتها كـ dependencies. أضف ما يلي إلى ملف _Cargo.toml_ لـ `hello_macro_derive`:

<Listing file-name="hello_macro_derive/Cargo.toml">

```toml
{{#include ../listings/ch20-advanced-features/listing-20-40/hello_macro/hello_macro_derive/Cargo.toml:6:12}}
```

</Listing>

لبدء تعريف الـ procedural macro، ضع الكود في القائمة 20-40 في ملف _src/lib.rs_ الخاص بك لـ crate `hello_macro_derive`. لاحظ أن هذا الكود لن يتم تجميعه حتى نضيف تعريفًا لـ function `impl_hello_macro`.

<Listing number="20-40" file-name="hello_macro_derive/src/lib.rs" caption="الكود الذي ستحتاجه معظم الـ procedural macro crates لمعالجة كود Rust">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-40/hello_macro/hello_macro_derive/src/lib.rs}}
```

</Listing>

لاحظ أننا قسمنا الكود إلى function `hello_macro_derive`، وهي المسؤولة عن parse الـ `TokenStream`، و function `impl_hello_macro`، وهي المسؤولة عن تحويل شجرة بناء الجملة (syntax tree): هذا يجعل كتابة procedural macro أكثر ملاءمة. سيكون الكود في الـ function الخارجية (`hello_macro_derive` في هذه الحالة) هو نفسه تقريبًا لكل procedural macro crate تراه أو تنشئه. سيكون الكود الذي تحدده في نص الـ function الداخلية (`impl_hello_macro` في هذه الحالة) مختلفًا اعتمادًا على الغرض من الـ procedural macro الخاص بك.

لقد قدمنا ثلاثة crates جديدة: `proc_macro`، و [`syn`][syn]، و [`quote`][quote]. يأتي الـ crate `proc_macro` مع Rust، لذلك لم نكن بحاجة إلى إضافته إلى الـ dependencies في _Cargo.toml_. الـ crate `proc_macro` هو واجهة برمجة تطبيقات (API) المترجم التي تسمح لنا بقراءة ومعالجة كود Rust من الكود الخاص بنا.

يقوم الـ crate `syn` بـ parse كود Rust من string إلى هيكل بيانات يمكننا إجراء عمليات عليه. يقوم الـ crate `quote` بتحويل هياكل بيانات `syn` مرة أخرى إلى كود Rust. تجعل هذه الـ crates من السهل جدًا parse أي نوع من كود Rust قد نرغب في التعامل معه: كتابة parser كامل لكود Rust ليست مهمة بسيطة.

سيتم استدعاء function `hello_macro_derive` عندما يحدد مستخدم مكتبتنا `#[derive(HelloMacro)]` على نوع. هذا ممكن لأننا أضفنا تعليقًا توضيحيًا لـ function `hello_macro_derive` هنا بـ `proc_macro_derive` وحددنا الاسم `HelloMacro`، الذي يطابق اسم الـ trait الخاص بنا؛ هذا هو الاصطلاح الذي تتبعه معظم الـ procedural macros.

تقوم function `hello_macro_derive` أولاً بتحويل الـ `input` من `TokenStream` إلى هيكل بيانات يمكننا بعد ذلك تفسيره وإجراء عمليات عليه. هذا هو المكان الذي يأتي فيه دور `syn`. تأخذ function `parse` في `syn` `TokenStream` وتُرجع struct `DeriveInput` يمثل كود Rust الذي تم عمل parse له. تُظهر القائمة 20-41 الأجزاء ذات الصلة من struct `DeriveInput` التي نحصل عليها من parse الـ string `struct Pancakes;`.

<Listing number="20-41" caption="مثيل `DeriveInput` الذي نحصل عليه عند parse الكود الذي يحتوي على attribute الـ macro في القائمة 20-37">

```rust,ignore
DeriveInput {
    // --snip--

    ident: Ident {
        ident: "Pancakes",
        span: #0 bytes(95..103)
    },
    data: Struct(
        DataStruct {
            struct_token: Struct,
            fields: Unit,
            semi_token: Some(
                Semi
            )
        }
    )
}
```

</Listing>

تُظهر fields هذا الـ struct أن كود Rust الذي قمنا بـ parse له هو unit struct مع الـ `ident` (*المعرف*، بمعنى الاسم) لـ `Pancakes`. هناك المزيد من الـ fields على هذا الـ struct لوصف جميع أنواع كود Rust؛ تحقق من [وثائق `syn` لـ `DeriveInput`][syn-docs] لمزيد من المعلومات.

قريبًا سنقوم بتعريف function `impl_hello_macro`، وهو المكان الذي سنقوم فيه ببناء كود Rust الجديد الذي نريد تضمينه. ولكن قبل أن نفعل ذلك، لاحظ أن الخرج لـ `derive` macro الخاص بنا هو أيضًا `TokenStream`. تتم إضافة الـ `TokenStream` المُرجع إلى الكود الذي يكتبه مستخدمو الـ crate الخاص بنا، لذلك عندما يقومون بـ compile الـ crate الخاص بهم، سيحصلون على الوظيفة الإضافية التي نوفرها في الـ `TokenStream` المعدل.

ربما لاحظت أننا نستدعي `unwrap` لجعل function `hello_macro_derive` يحدث لها panic! إذا فشل استدعاء function `syn::parse` هنا. من الضروري أن يحدث panic! لـ procedural macro الخاص بنا عند حدوث أخطاء لأن functions `proc_macro_derive` يجب أن تُرجع `TokenStream` بدلاً من `Result` لتتوافق مع API الـ procedural macro. لقد قمنا بتبسيط هذا المثال باستخدام `unwrap`؛ في كود الإنتاج، يجب عليك توفير رسائل خطأ أكثر تحديدًا حول ما حدث بشكل خاطئ باستخدام `panic!` أو `expect`.

الآن بعد أن أصبح لدينا الكود لتحويل كود Rust المشروح من `TokenStream` إلى مثيل `DeriveInput`، دعنا ننشئ الكود الذي يطبق الـ trait `HelloMacro` على النوع المشروح، كما هو موضح في القائمة 20-42.

<Listing number="20-42" file-name="hello_macro_derive/src/lib.rs" caption="تطبيق trait `HelloMacro` باستخدام كود Rust الذي تم عمل parse له">

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-42/hello_macro/hello_macro_derive/src/lib.rs:here}}
```

</Listing>

نحصل على مثيل struct `Ident` يحتوي على اسم (المعرف) النوع المشروح باستخدام `ast.ident`. يُظهر الـ struct في القائمة 20-41 أنه عندما نقوم بتشغيل function `impl_hello_macro` على الكود في القائمة 20-37، فإن الـ `ident` الذي نحصل عليه سيكون له الـ field `ident` بقيمة `"Pancakes"`. وبالتالي، سيحتوي المتغير `name` في القائمة 20-42 على مثيل struct `Ident`، والذي عند طباعته، سيكون الـ string `"Pancakes"`، وهو اسم الـ struct في القائمة 20-37.

يسمح لنا الـ macro `quote!` بتعريف كود Rust الذي نريد إرجاعه. يتوقع المترجم شيئًا مختلفًا عن النتيجة المباشرة لتنفيذ الـ macro `quote!`، لذلك نحتاج إلى تحويله إلى `TokenStream`. نقوم بذلك عن طريق استدعاء الـ method `into`، الذي يستهلك هذا التمثيل الوسيط ويُرجع قيمة من النوع `TokenStream` المطلوب.

يوفر الـ macro `quote!` أيضًا بعض آليات القوالب الرائعة: يمكننا إدخال `#name`، وسيحل محله `quote!` بالقيمة الموجودة في المتغير `name`. يمكنك حتى القيام ببعض التكرار المشابه للطريقة التي تعمل بها الـ macros العادية. تحقق من [وثائق الـ crate `quote`][quote-docs] للحصول على مقدمة شاملة.

نريد أن يقوم الـ procedural macro الخاص بنا بإنشاء تطبيق لـ trait `HelloMacro` الخاص بنا للنوع الذي قام المستخدم بشرحه، والذي يمكننا الحصول عليه باستخدام `#name`. يحتوي تطبيق الـ trait على function واحدة `hello_macro`، التي يحتوي نصها على الوظيفة التي نريد توفيرها: طباعة `Hello, Macro! My name is` ثم اسم النوع المشروح.

الـ macro `stringify!` المستخدم هنا مدمج في Rust. يأخذ تعبير Rust، مثل `1 + 2`، وفي الـ compile time يحول التعبير إلى string literal، مثل `"1 + 2"`. هذا يختلف عن `format!` أو `println!`، وهما macros يقومان بتقييم التعبير ثم تحويل النتيجة إلى `String`. هناك احتمال أن يكون الـ input `#name` تعبيرًا للطباعة حرفيًا، لذلك نستخدم `stringify!`. يوفر استخدام `stringify!` أيضًا تخصيصًا عن طريق تحويل `#name` إلى string literal في الـ compile time.

في هذه المرحلة، يجب أن يكتمل `cargo build` بنجاح في كل من `hello_macro` و `hello_macro_derive`. دعنا نربط هذه الـ crates بالكود في القائمة 20-37 لرؤية الـ procedural macro في العمل! قم بإنشاء مشروع ثنائي جديد في دليل _projects_ الخاص بك باستخدام `cargo new pancakes`. نحتاج إلى إضافة `hello_macro` و `hello_macro_derive` كـ dependencies في _Cargo.toml_ الخاص بـ crate `pancakes`. إذا كنت تنشر إصداراتك من `hello_macro` و `hello_macro_derive` على [crates.io](https://crates.io/)، فستكون dependencies عادية؛ إذا لم يكن كذلك، يمكنك تحديدها كـ dependencies `path` على النحو التالي:

```toml
{{#include ../listings/ch20-advanced-features/no-listing-21-pancakes/pancakes/Cargo.toml:6:8}}
```

ضع الكود في القائمة 20-37 في _src/main.rs_، وقم بتشغيل `cargo run`: يجب أن يطبع `Hello, Macro! My name is Pancakes!`. تم تضمين تطبيق الـ trait `HelloMacro` من الـ procedural macro دون أن يحتاج الـ crate `pancakes` إلى تطبيقه؛ أضاف `#[derive(HelloMacro)]` تطبيق الـ trait.

بعد ذلك، دعنا نستكشف كيف تختلف الأنواع الأخرى من الـ procedural macros عن الـ custom `derive` macros.

### الـ Attribute-Like Macros

الـ attribute-like macros مشابهة لـ custom `derive` macros، ولكن بدلاً من إنشاء كود لـ attribute `derive`، فإنها تسمح لك بإنشاء attributes جديدة. إنها أيضًا أكثر مرونة: يعمل `derive` فقط مع الـ structs والـ enums؛ يمكن تطبيق الـ attributes على عناصر أخرى أيضًا، مثل الـ functions. إليك مثال على استخدام attribute-like macro. لنفترض أن لديك attribute يسمى `route` يشرح الـ functions عند استخدام إطار عمل لتطبيق ويب:

```rust,ignore
#[route(GET, "/")]
fn index() {
```

سيتم تعريف attribute `#[route]` هذا بواسطة إطار العمل كـ procedural macro. سيبدو توقيع function تعريف الـ macro كما يلي:

```rust,ignore
#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {
```

هنا، لدينا معلمات من النوع `TokenStream`. الأول هو لمحتويات الـ attribute: الجزء `GET, "/"`. والثاني هو نص العنصر المرفق به الـ attribute: في هذه الحالة، `fn index() {}` وبقية نص الـ function.

بخلاف ذلك، تعمل الـ attribute-like macros بنفس طريقة عمل الـ custom `derive` macros: تقوم بإنشاء crate بنوع الـ crate `proc-macro` وتطبيق function ينشئ الكود الذي تريده!

### الـ Function-Like Macros

الـ function-like macros هي الشكل الثالث من الـ procedural macros. على عكس الـ declarative macros، التي تعمل عن طريق المطابقة مع الـ patterns، تعمل الـ function-like macros عن طريق أخذ `TokenStream` كمدخل وإرجاع `TokenStream` كخرج. على سبيل المثال، يمكن أن يكون لديك macro يسمى `sql!` يقوم بـ parse محتويات string ويتحقق من صحة بناء جملة SQL داخله.

```rust,ignore
let sql = sql!(SELECT * FROM posts WHERE id=1);
```

سيبدو توقيع function تعريف الـ macro كما يلي:

```rust,ignore
#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
```

هذا مشابه لـ custom `derive` macros، ولكن بدلاً من أن يكون له attribute يحدد نوع الـ macro، فإن function تعريف الـ macro لها attribute `#[proc_macro]`، وتأخذ `TokenStream` واحدًا كمدخل.

[ref]: https://doc.rust-lang.org/reference/macros-by-example.html
[tlborm]: https://danielkeep.github.io/tlborm/book/
[syn]: https://crates.io/crates/syn
[quote]: https://crates.io/crates/quote
[syn-docs]: https://docs.rs/syn/latest/syn/struct.DeriveInput.html
[quote-docs]: https://docs.rs/quote/latest/quote/
