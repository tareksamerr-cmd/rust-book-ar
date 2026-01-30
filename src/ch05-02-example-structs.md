## برنامج مثال يستخدم التركيبات (Structs)

لفهم متى قد نرغب في استخدام Structs، دعنا نكتب برنامجًا يحسب مساحة مستطيل. سنبدأ باستخدام متغيرات مفردة ثم نقوم بـ إعادة الهيكلة (Refactoring) للبرنامج حتى نستخدم Structs بدلاً من ذلك.

دعنا ننشئ مشروعًا ثنائيًا جديدًا باستخدام Cargo يسمى _rectangles_ والذي سيأخذ العرض والارتفاع لمستطيل محدد بالبكسل ويحسب مساحة المستطيل. توضح القائمة 5-8 برنامجًا قصيرًا بإحدى طرق القيام بذلك بالضبط في ملف _src/main.rs_ الخاص بمشروعنا.

<Listing number="5-8" file-name="src/main.rs" caption="Calculating the area of a rectangle specified by separate width and height variables">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/src/main.rs:all}}
```

</Listing>

الآن، قم بتشغيل هذا البرنامج باستخدام `cargo run`:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/output.txt}}
```

ينجح هذا الكود في معرفة مساحة المستطيل عن طريق استدعاء الدالة `area` بكل بُعد، ولكن يمكننا فعل المزيد لجعل هذا الكود واضحًا وقابلًا للقراءة.

تظهر المشكلة في هذا الكود واضحة في توقيع `area`:

```rust,ignore
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/src/main.rs:here}}
```

من المفترض أن تحسب الدالة `area` مساحة مستطيل واحد، لكن الدالة التي كتبناها تحتوي على معاملين، وليس من الواضح في أي مكان في برنامجنا أن المعاملين مرتبطان. سيكون تجميع width و height معًا أكثر قابلية للقراءة وأسهل في الإدارة. لقد ناقشنا بالفعل إحدى الطرق التي قد نفعل بها ذلك في قسم [“The Tuple Type”][the-tuple-type]<!-- ignore --> من الفصل 3: باستخدام الصفوف (Tuples).

### Refactoring باستخدام Tuples

توضح القائمة 5-9 إصدارًا آخر من برنامجنا يستخدم Tuples.

<Listing number="5-9" file-name="src/main.rs" caption="Specifying the width and height of the rectangle with a tuple">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-09/src/main.rs}}
```

</Listing>

بطريقة ما، هذا البرنامج أفضل. تسمح لنا Tuples بإضافة القليل من الهيكلة، ونحن الآن نمرر وسيطًا واحدًا فقط. ولكن بطريقة أخرى، هذا الإصدار أقل وضوحًا: لا تسمي Tuples عناصرها، لذلك يجب علينا الفهرسة في أجزاء Tuple، مما يجعل حسابنا أقل وضوحًا.

لن يهم خلط width و height في حساب المساحة، ولكن إذا أردنا رسم المستطيل على الشاشة، فسيكون الأمر مهمًا! سيتعين علينا أن نضع في اعتبارنا أن width هو فهرس Tuple رقم `0` وأن height هو فهرس Tuple رقم `1`. سيكون هذا أصعب على شخص آخر اكتشافه ووضعه في الاعتبار إذا كان سيستخدم الكود الخاص بنا. نظرًا لأننا لم ننقل معنى بياناتنا في الكود الخاص بنا، فقد أصبح من الأسهل الآن إدخال أخطاء.

<!-- Old headings. Do not remove or links may break. -->

<a id="refactoring-with-structs-adding-more-meaning"></a>

### Refactoring باستخدام Structs

نستخدم Structs لإضافة معنى عن طريق تسمية البيانات. يمكننا تحويل Tuple الذي نستخدمه إلى Struct باسم للكل بالإضافة إلى أسماء للأجزاء، كما هو موضح في القائمة 5-10.

<Listing number="5-10" file-name="src/main.rs" caption="Defining a `Rectangle` struct">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-10/src/main.rs}}
```

</Listing>

هنا، قمنا بتعريف Struct وسميناها `Rectangle`. داخل الأقواس المعقوفة، قمنا بتعريف الحقول (Fields) كـ `width` و `height`، وكلاهما من النوع `u32`. بعد ذلك، في `main`، أنشأنا مثيلًا (instance) معينًا من `Rectangle` بعرض `30` وارتفاع `50`.

تم تعريف الدالة `area` الآن بمعامل واحد، سميناه `rectangle`، ونوعه هو استعارة غير قابلة للتغيير (immutable borrow) لمثيل Struct `Rectangle`. كما ذكرنا في الفصل 4، نريد استعارة (Borrow) Struct بدلاً من أخذ الملكية (Ownership) لها. بهذه الطريقة، تحتفظ `main` بـ Ownership الخاصة بها ويمكنها الاستمرار في استخدام `rect1`، وهذا هو السبب في أننا نستخدم `&` في توقيع الدالة وعندما نستدعي الدالة.

تصل الدالة `area` إلى Fields `width` و `height` لمثيل `Rectangle` (لاحظ أن الوصول إلى Fields لمثيل Struct مستعار لا ينقل قيم Field، ولهذا السبب ترى غالبًا عمليات Borrow لـ Structs). يقول توقيع الدالة `area` الآن بالضبط ما نعنيه: احسب مساحة `Rectangle`، باستخدام Fields `width` و `height` الخاصة بها. هذا ينقل أن width و height مرتبطان ببعضهما البعض، ويعطي أسماء وصفية للقيم بدلاً من استخدام قيم فهرس Tuple `0` و `1`. هذا مكسب للوضوح.

<!-- Old headings. Do not remove or links may break. -->

<a id="adding-useful-functionality-with-derived-traits"></a>

### إضافة وظائف باستخدام السمات المشتقة (Derived Traits)

سيكون من المفيد أن نتمكن من طباعة مثيل `Rectangle` أثناء تصحيح الأخطاء (Debug) في برنامجنا ورؤية القيم لجميع Fields الخاصة به. تحاول القائمة 5-11 استخدام الماكرو [`println!`][println]<!-- ignore --> كما استخدمناه في الفصول السابقة. ومع ذلك، لن ينجح هذا.

<Listing number="5-11" file-name="src/main.rs" caption="Attempting to print a `Rectangle` instance">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/src/main.rs}}
```

</Listing>

عندما نقوم بتجميع هذا الكود، نحصل على خطأ بهذه الرسالة الأساسية:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/output.txt:3}}
```

يمكن لـ ماكرو `println!` القيام بالعديد من أنواع التنسيق، وبشكل افتراضي، تخبر الأقواس المعقوفة `println!` باستخدام تنسيق يُعرف باسم `Display`: إخراج مخصص للاستهلاك المباشر للمستخدم النهائي. تطبق الأنواع البدائية (primitive types) التي رأيناها حتى الآن `Display` افتراضيًا لأنه لا توجد سوى طريقة واحدة تريد بها إظهار `1` أو أي نوع بدائي آخر للمستخدم. ولكن مع Structs، فإن الطريقة التي يجب أن يقوم بها `println!` بتنسيق الإخراج أقل وضوحًا لأن هناك المزيد من إمكانيات العرض: هل تريد فواصل أم لا؟ هل تريد طباعة الأقواس المعقوفة؟ هل يجب عرض جميع Fields؟ بسبب هذا الغموض، لا يحاول Rust تخمين ما نريده، ولا تحتوي Structs على تطبيق متوفر لـ `Display` لاستخدامه مع `println!` والعنصر النائب `{}`.

إذا واصلنا قراءة الأخطاء، فسنجد هذه الملاحظة المفيدة:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/output.txt:9:10}}
```

دعنا نجربها! سيبدو استدعاء ماكرو `println!` الآن كما يلي: `println!("rect1 is {rect1:?}");`. وضع المحدد `:?` داخل الأقواس المعقوفة يخبر `println!` أننا نريد استخدام تنسيق إخراج يسمى `Debug`. تُمكّننا السمة (Trait) `Debug` من طباعة Struct الخاص بنا بطريقة مفيدة للمطورين حتى نتمكن من رؤية قيمتها أثناء قيامنا بـ Debug الكود الخاص بنا.

قم بتجميع الكود بهذا التغيير. يا للأسف! ما زلنا نحصل على خطأ:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-01-debug/output.txt:3}}
```

ولكن مرة أخرى، يعطينا المترجم ملاحظة مفيدة:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-01-debug/output.txt:9:10}}
```

يتضمن Rust وظيفة لطباعة معلومات Debug، ولكن يجب علينا الاشتراك صراحةً لجعل هذه الوظيفة متاحة لـ Struct الخاص بنا. للقيام بذلك، نضيف السمة الخارجية (Attribute) `#[derive(Debug)]` قبل تعريف Struct مباشرةً، كما هو موضح في القائمة 5-12.

<Listing number="5-12" file-name="src/main.rs" caption="Adding the attribute to derive the `Debug` trait and printing the `Rectangle` instance using debug formatting">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-12/src/main.rs}}
```

</Listing>

الآن عندما نقوم بتشغيل البرنامج، لن نحصل على أي أخطاء، وسنرى الإخراج التالي:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-12/output.txt}}
```

جيد! إنه ليس أجمل إخراج، ولكنه يظهر قيم جميع Fields لهذا المثيل، مما سيساعد بالتأكيد أثناء Debug. عندما يكون لدينا Structs أكبر، من المفيد أن يكون لدينا إخراج أسهل قليلاً في القراءة؛ في هذه الحالات، يمكننا استخدام ```````{:#?}``````` بدلاً من `{:?}` في سلسلة `println!`. في هذا المثال، سيؤدي استخدام نمط ```````{:#?}``````` إلى إخراج ما يلي:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-02-pretty-debug/output.txt}}
```

هناك طريقة أخرى لطباعة قيمة باستخدام تنسيق `Debug` وهي استخدام الماكرو [`dbg!`][dbg]<!-- ignore -->، الذي يأخذ Ownership لتعبير (على عكس `println!`، الذي يأخذ reference)، ويطبع الملف ورقم السطر الذي يحدث فيه استدعاء ماكرو `dbg!` في الكود الخاص بك جنبًا إلى جنب مع القيمة الناتجة لهذا التعبير، ويعيد Ownership للقيمة.

> ملاحظة: يطبع استدعاء ماكرو `dbg!` إلى مجرى وحدة التحكم للخطأ القياسي (`stderr`)، على عكس `println!`، الذي يطبع إلى مجرى وحدة التحكم للإخراج القياسي (`stdout`). سنتحدث أكثر عن `stderr` و `stdout` في قسم [“Redirecting Errors to Standard Error” section in Chapter 12][err]<!-- ignore -->.

إليك مثال حيث نهتم بالقيمة التي يتم تعيينها لـ Field `width`، بالإضافة إلى قيمة Struct بأكمله في `rect1`:

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-05-dbg-macro/src/main.rs}}
```

يمكننا وضع `dbg!` حول التعبير `30 * scale`، ولأن `dbg!` يعيد Ownership لقيمة التعبير، سيحصل Field `width` على نفس القيمة كما لو لم يكن لدينا استدعاء `dbg!` هناك. لا نريد أن يأخذ `dbg!` Ownership لـ `rect1`، لذلك نستخدم reference لـ `rect1` في الاستدعاء التالي. إليك كيف يبدو إخراج هذا المثال:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/no-listing-05-dbg-macro/output.txt}}
```

يمكننا أن نرى أن الجزء الأول من الإخراج جاء من السطر 10 في _src/main.rs_ حيث نقوم بـ Debug التعبير `30 * scale`، وقيمته الناتجة هي `60` (تنسيق `Debug` المطبق للأعداد الصحيحة هو طباعة قيمتها فقط). يخرج استدعاء `dbg!` في السطر 14 من _src/main.rs_ قيمة `&rect1`، وهي Struct `Rectangle`. يستخدم هذا الإخراج تنسيق `Debug` الجميل لنوع `Rectangle`. يمكن أن يكون ماكرو `dbg!` مفيدًا حقًا عندما تحاول معرفة ما يفعله الكود الخاص بك!

بالإضافة إلى Trait `Debug`، وفر Rust عددًا من Traits لنا لاستخدامها مع Attribute `derive` والتي يمكن أن تضيف سلوكًا مفيدًا لأنواعنا المخصصة. يتم سرد هذه Traits وسلوكياتها في [Appendix C][app-c]<!-- ignore -->. سنغطي كيفية تطبيق هذه Traits بسلوك مخصص بالإضافة إلى كيفية إنشاء Traits الخاصة بك في الفصل 10. هناك أيضًا العديد من Attributes بخلاف `derive`؛ لمزيد من المعلومات، راجع [the “Attributes” section of the Rust Reference][attributes].

الدالة `area` الخاصة بنا محددة للغاية: إنها تحسب مساحة المستطيلات فقط. سيكون من المفيد ربط هذا السلوك بشكل أوثق بـ Struct `Rectangle` الخاص بنا لأنه لن يعمل مع أي نوع آخر. دعنا نلقي نظرة على كيف يمكننا الاستمرار في Refactoring هذا الكود عن طريق تحويل الدالة `area` إلى دالة عضو (Method) `area` معرفة على نوع `Rectangle` الخاص بنا.

[the-tuple-type]: ch03-02-data-types.html#the-tuple-type
[app-c]: appendix-03-derivable-traits.md
[println]: ../std/macro.println.html
[dbg]: ../std/macro.dbg.html
[err]: ch12-06-writing-to-stderr-instead-of-stdout.html
[attributes]: ../reference/attributes.html
