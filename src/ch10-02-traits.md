<!-- Old headings. Do not remove or links may break. -->

<a id="traits-defining-shared-behavior"></a>

## تعريف السلوك المشترك باستخدام السمات (Defining Shared Behavior with Traits)

تحدد الـ سمة (trait) الوظائف التي يمتلكها نوع معين ويمكنه مشاركتها مع أنواع أخرى. يمكننا استخدام الـ traits لتعريف السلوك المشترك بطريقة مجردة (abstract). كما يمكننا استخدام قيود السمات (trait bounds) لتحديد أن النوع العام (generic type) يمكن أن يكون أي نوع يمتلك سلوكًا معينًا.

> ملاحظة: الـ traits تشبه ميزة تسمى غالبًا الواجهات (interfaces) في لغات أخرى، وإن كان مع بعض الاختلافات.

### تعريف سمة (Defining a Trait)

يتكون سلوك النوع من الدوال (methods) التي يمكننا استدعاؤها على ذلك النوع. تتشارك الأنواع المختلفة في نفس السلوك إذا كان بإمكاننا استدعاء نفس الـ methods على كل تلك الأنواع. تعريفات الـ Trait هي وسيلة لتجميع تواقيع الدوال (method signatures) معًا لتعريف مجموعة من السلوكيات الضرورية لتحقيق غرض ما.

على سبيل المثال، لنفترض أن لدينا عدة هياكل (structs) تحمل أنواعًا وكميات مختلفة من النصوص: هيكل `NewsArticle` يحمل قصة إخبارية مسجلة في موقع معين، وهيكل `SocialPost` يمكن أن يحتوي على 280 حرفًا كحد أقصى مع بيانات وصفية (metadata) تشير إلى ما إذا كان منشورًا جديدًا، أو إعادة نشر، أو ردًا على منشور آخر.

نريد إنشاء حزمة مكتبة (library crate) لمجمع وسائط تسمى `aggregator` يمكنها عرض ملخصات للبيانات التي قد تكون مخزنة في مثيل (instance) من `NewsArticle` أو `SocialPost`. للقيام بذلك، نحتاج إلى ملخص من كل نوع، وسنطلب ذلك الملخص عن طريق استدعاء دالة `summarize` على الـ instance. توضح القائمة 10-12 تعريف سمة `Summary` عامة (public) تعبر عن هذا السلوك.

<Listing number="10-12" file-name="src/lib.rs" caption="سمة `Summary` تتكون من السلوك الذي توفره دالة `summarize` ">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-12/src/lib.rs}}
```

</Listing>

هنا، نعلن عن trait باستخدام الكلمة المفتاحية `trait` ثم اسم الـ trait، وهو `Summary` في هذه الحالة. نعلن أيضًا عن الـ trait كـ `pub` بحيث يمكن للـ crates التي تعتمد على هذه الـ crate استخدام هذه الـ trait أيضًا، كما سنرى في بضعة أمثلة. داخل الأقواس المتعرجة، نعلن عن الـ method signatures التي تصف سلوكيات الأنواع التي تنفذ (implement) هذه الـ trait، وهي في هذه الحالة `fn summarize(&self) -> String`.

بعد الـ method signature، وبدلاً من توفير تنفيذ (implementation) داخل أقواس متعرجة، نستخدم فاصلة منقوطة. يجب على كل نوع ينفذ هذه الـ trait توفير سلوكه المخصص لجسم الـ method. سيفرض المترجم (compiler) أن أي نوع يمتلك الـ `Summary` trait سيكون لديه الـ method المسمى `summarize` معرفًا بهذا التوقيع (signature) تمامًا.

يمكن أن يحتوي الـ trait على عدة methods في جسمه: يتم إدراج الـ method signatures واحدًا تلو الآخر في كل سطر، وينتهي كل سطر بفاصلة منقوطة.

### تنفيذ سمة على نوع (Implementing a Trait on a Type)

الآن بعد أن حددنا التواقيع المطلوبة لـ methods سمة `Summary` يمكننا تنفيذها على الأنواع في مجمع الوسائط الخاص بنا. توضح القائمة 10-13 تنفيذًا لـ `Summary` trait على هيكل `NewsArticle` يستخدم العنوان الرئيسي، والمؤلف، والموقع لإنشاء القيمة المعادة من `summarize`. بالنسبة لهيكل `SocialPost` نحدد `summarize` كاسم المستخدم متبوعًا بالنص الكامل للمنشور، بافتراض أن محتوى المنشور محدود بالفعل بـ 280 حرفًا.

<Listing number="10-13" file-name="src/lib.rs" caption="تنفيذ سمة `Summary` على نوعي `NewsArticle` و `SocialPost` ">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-13/src/lib.rs:here}}
```

</Listing>

تنفيذ trait على نوع يشبه تنفيذ الـ methods العادية. الفرق هو أنه بعد `impl` نضع اسم الـ trait الذي نريد تنفيذه، ثم نستخدم الكلمة المفتاحية `for` ثم نحدد اسم النوع الذي نريد تنفيذ الـ trait له. داخل كتلة `impl` نضع الـ method signatures التي حددها تعريف الـ trait. وبدلاً من إضافة فاصلة منقوطة بعد كل signature، نستخدم الأقواس المتعرجة ونملأ جسم الـ method بالسلوك المحدد الذي نريده لـ methods الـ trait لهذا النوع المعين.

الآن بعد أن نفذت المكتبة الـ `Summary` trait على `NewsArticle` و `SocialPost` يمكن لمستخدمي الـ crate استدعاء الـ trait methods على instances من `NewsArticle` و `SocialPost` بنفس الطريقة التي نستدعي بها الـ methods العادية. الفرق الوحيد هو أنه يجب على المستخدم إحضار الـ trait إلى النطاق (scope) بالإضافة إلى الأنواع. إليك مثال على كيفية استخدام binary crate لمكتبة `aggregator` الخاصة بنا:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-01-calling-trait-method/src/main.rs}}
```

يطبع هذا الكود `1 new post: horse_ebooks: of course, as you probably already know, people`.

يمكن للـ crates الأخرى التي تعتمد على `aggregator` crate أيضًا إحضار الـ `Summary` trait إلى الـ scope لتنفيذ `Summary` على أنواعها الخاصة. أحد القيود التي يجب ملاحظتها هو أنه يمكننا تنفيذ trait على نوع فقط إذا كان الـ trait أو النوع، أو كلاهما، محليًا (local) للـ crate الخاصة بنا. على سبيل المثال، يمكننا تنفيذ traits المكتبة القياسية (standard library) مثل `Display` على نوع مخصص مثل `SocialPost` كجزء من وظائف `aggregator` crate لأن النوع `SocialPost` محلي لـ `aggregator` crate الخاصة بنا. يمكننا أيضًا تنفيذ `Summary` على `Vec<T>` في `aggregator` crate لأن الـ trait المسمى `Summary` محلي لـ `aggregator` crate الخاصة بنا.

لكن لا يمكننا تنفيذ traits خارجية على أنواع خارجية. على سبيل المثال، لا يمكننا تنفيذ الـ `Display` trait على `Vec<T>` داخل `aggregator` crate الخاصة بنا، لأن `Display` و `Vec<T>` كلاهما معرفان في المكتبة القياسية وليسا محليين لـ `aggregator` crate الخاصة بنا. هذا القيد هو جزء من خاصية تسمى التماسك (coherence)، وبشكل أكثر تحديدًا قاعدة اليتيم (orphan rule)، وسميت بهذا الاسم لأن النوع الأب غير موجود. تضمن هذه القاعدة أن كود الأشخاص الآخرين لا يمكنه كسر كودك والعكس صحيح. بدون هذه القاعدة، يمكن لـ crates اثنتين تنفيذ نفس الـ trait لنفس النوع، ولن يعرف Rust أي تنفيذ يستخدم.

<!-- Old headings. Do not remove or links may break. -->

<a id="default-implementations"></a>

### استخدام التنفيذات الافتراضية (Using Default Implementations)

أحيانًا يكون من المفيد الحصول على سلوك افتراضي (default behavior) لبعض أو كل الـ methods في trait بدلاً من طلب تنفيذات لجميع الـ methods على كل نوع. بعد ذلك، عندما ننفذ الـ trait على نوع معين، يمكننا الاحتفاظ بالسلوك الافتراضي لكل method أو تجاوزه (override).

في القائمة 10-14، نحدد سلسلة نصية افتراضية لدالة `summarize` في سمة `Summary` بدلاً من الاكتفاء بتعريف الـ method signature، كما فعلنا في القائمة 10-12.

<Listing number="10-14" file-name="src/lib.rs" caption="تعريف سمة `Summary` مع تنفيذ افتراضي لدالة `summarize` ">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-14/src/lib.rs:here}}
```

</Listing>

لاستخدام تنفيذ افتراضي لتلخيص instances من `NewsArticle` نحدد كتلة `impl` فارغة باستخدام `impl Summary for NewsArticle {}`.

على الرغم من أننا لم نعد نعرف دالة `summarize` على `NewsArticle` مباشرة، فقد قدمنا تنفيذًا افتراضيًا وحددنا أن `NewsArticle` ينفذ الـ `Summary` trait. ونتيجة لذلك، لا يزال بإمكاننا استدعاء دالة `summarize` على instance من `NewsArticle` بهذا الشكل:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-02-calling-default-impl/src/main.rs:here}}
```

يطبع هذا الكود `New article available! (Read more...)`.

إنشاء تنفيذ افتراضي لا يتطلب منا تغيير أي شيء في تنفيذ `Summary` على `SocialPost` في القائمة 10-13. والسبب هو أن بناء الجملة (syntax) لتجاوز تنفيذ افتراضي هو نفسه الـ syntax لتنفيذ trait method ليس له تنفيذ افتراضي.

يمكن للتنفيذات الافتراضية استدعاء methods أخرى في نفس الـ trait، حتى لو لم يكن لتلك الـ methods الأخرى تنفيذ افتراضي. بهذه الطريقة، يمكن لـ trait توفير الكثير من الوظائف المفيدة وتتطلب فقط من المنفذين تحديد جزء صغير منها. على سبيل المثال، يمكننا تعريف الـ `Summary` trait ليكون له دالة `summarize_author` التي يكون تنفيذها مطلوبًا، ثم تعريف دالة `summarize` لها تنفيذ افتراضي يستدعي دالة `summarize_author`:

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/lib.rs:here}}
```

لاستخدام هذه النسخة من `Summary` نحتاج فقط إلى تعريف `summarize_author` عندما ننفذ الـ trait على نوع ما:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/lib.rs:impl}}
```

بعد تعريف `summarize_author` يمكننا استدعاء `summarize` على instances من هيكل `SocialPost` وسيقوم التنفيذ الافتراضي لـ `summarize` باستدعاء تعريف `summarize_author` الذي قدمناه. نظرًا لأننا نفذنا `summarize_author` فقد منحتنا الـ `Summary` trait سلوك دالة `summarize` دون مطالبتنا بكتابة أي كود إضافي. إليك كيف يبدو ذلك:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/main.rs:here}}
```

يطبع هذا الكود `1 new post: (Read more from @horse_ebooks...)`.

لاحظ أنه ليس من الممكن استدعاء التنفيذ الافتراضي من تنفيذ متجاوز (overriding implementation) لنفس الـ method.

<!-- Old headings. Do not remove or links may break. -->

<a id="traits-as-parameters"></a>

### استخدام السمات كمعاملات (Using Traits as Parameters)

الآن بعد أن عرفت كيفية تعريف وتنفيذ الـ traits، يمكننا استكشاف كيفية استخدام الـ traits لتعريف دوال تقبل العديد من الأنواع المختلفة. سنستخدم الـ `Summary` trait الذي نفذناه على نوعي `NewsArticle` و `SocialPost` في القائمة 10-13 لتعريف دالة `notify` تستدعي دالة `summarize` على معاملها (parameter) المسمى `item` وهو من نوع ما ينفذ الـ `Summary` trait. للقيام بذلك، نستخدم الـ `impl Trait` syntax بهذا الشكل:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-04-traits-as-parameters/src/lib.rs:here}}
```

بدلاً من نوع ملموس (concrete type) للمعامل `item` نحدد الكلمة المفتاحية `impl` واسم الـ trait. يقبل هذا الـ parameter أي نوع ينفذ الـ trait المحدد. في جسم `notify` يمكننا استدعاء أي methods على `item` تأتي من الـ `Summary` trait، مثل `summarize`. يمكننا استدعاء `notify` وتمرير أي instance من `NewsArticle` أو `SocialPost`. الكود الذي يستدعي الدالة مع أي نوع آخر، مثل `String` أو `i32` لن يترجم، لأن تلك الأنواع لا تنفذ `Summary`.

<!-- Old headings. Do not remove or links may break. -->

<a id="fixing-the-largest-function-with-trait-bounds"></a>

#### بناء جملة قيد السمة (Trait Bound Syntax)

يعمل الـ `impl Trait` syntax في الحالات المباشرة ولكنه في الواقع عبارة عن اختصار برمجي (syntax sugar) لشكل أطول يعرف باسم قيد السمة (trait bound)؛ ويبدو هكذا:

```rust,ignore
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```

هذا الشكل الأطول مكافئ للمثال في القسم السابق ولكنه أكثر تفصيلاً. نضع الـ trait bounds مع الإعلان عن معامل النوع العام (generic type parameter) بعد نقطتين وداخل أقواس زاوية.

يعتبر الـ trait bound syntax مناسبًا للحالات الأكثر تعقيدًا. على سبيل المثال، يمكن أن يكون لدينا معاملان ينفذان `Summary`. القيام بذلك باستخدام الـ `impl Trait` syntax يبدو هكذا:

```rust,ignore
pub fn notify(item1: &impl Summary, item2: &impl Summary) {
```

استخدام `impl Trait` مناسب إذا أردنا أن تسمح هذه الدالة لـ `item1` و `item2` بأن يكون لهما نوعان مختلفان (طالما أن كلا النوعين ينفذان `Summary`). أما إذا أردنا إجبار كلا المعاملين على أن يكون لهما نفس النوع، فيجب علينا استخدام trait bound بهذا الشكل:

```rust,ignore
pub fn notify<T: Summary>(item1: &T, item2: &T) {
```

النوع العام `T` المحدد كنوع للمعاملين `item1` و `item2` يقيد الدالة بحيث يجب أن يكون النوع الملموس للقيمة الممررة كوسيط (argument) لـ `item1` و `item2` هو نفسه.

<!-- Old headings. Do not remove or links may break. -->

<a id="specifying-multiple-trait-bounds-with-the--syntax"></a>

#### قيود سمات متعددة باستخدام الـ `+` Syntax (Multiple Trait Bounds with the `+` Syntax)

يمكننا أيضًا تحديد أكثر من trait bound واحد. لنفترض أننا أردنا أن تستخدم `notify` تنسيق العرض (display formatting) بالإضافة إلى `summarize` على `item`: نحدد في تعريف `notify` أن `item` يجب أن ينفذ كلاً من `Display` و `Summary`. يمكننا القيام بذلك باستخدام الـ `+` syntax:

```rust,ignore
pub fn notify(item: &(impl Summary + Display)) {
```

الـ `+` syntax صالح أيضًا مع الـ trait bounds على الأنواع العامة:

```rust,ignore
pub fn notify<T: Summary + Display>(item: &T) {
```

مع تحديد قيدي السمة، يمكن لجسم `notify` استدعاء `summarize` واستخدام `{}` لتنسيق `item`.

#### قيود سمات أكثر وضوحًا باستخدام بنود `where` (Clearer Trait Bounds with `where` Clauses)

استخدام الكثير من الـ trait bounds له سلبياته. فكل نوع عام له الـ trait bounds الخاصة به، لذا فإن الدوال التي تحتوي على عدة معاملات أنواع عامة يمكن أن تحتوي على الكثير من معلومات الـ trait bound بين اسم الدالة وقائمة معاملاتها، مما يجعل توقيع الدالة (function signature) صعب القراءة. لهذا السبب، يمتلك Rust بناء جملة بديلاً لتحديد الـ trait bounds داخل بند `where` بعد function signature. لذا، بدلاً من كتابة هذا:

```rust,ignore
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {
```

يمكننا استخدام بند `where` بهذا الشكل:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-07-where-clause/src/lib.rs:here}}
```

توقيع هذه الدالة أقل فوضى: اسم الدالة، وقائمة المعاملات، ونوع الإرجاع قريبة من بعضها البعض، بشكل مشابه لدالة لا تحتوي على الكثير من الـ trait bounds.

### إرجاع الأنواع التي تنفذ السمات (Returning Types That Implement Traits)

يمكننا أيضًا استخدام الـ `impl Trait` syntax في موضع الإرجاع لإعادة قيمة من نوع ما ينفذ trait، كما هو موضح هنا:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-05-returning-impl-trait/src/lib.rs:here}}
```

باستخدام `impl Summary` لنوع الإرجاع، نحدد أن دالة `returns_summarizable` تعيد نوعًا ما ينفذ الـ `Summary` trait دون تسمية النوع الملموس. في هذه الحالة، تعيد `returns_summarizable` هيكل `SocialPost` ولكن الكود الذي يستدعي هذه الدالة لا يحتاج إلى معرفة ذلك.

القدرة على تحديد نوع الإرجاع فقط من خلال الـ trait الذي ينفذه مفيدة بشكل خاص في سياق الـ closures والـ iterators، والتي نغطيها في الفصل 13. تنشئ الـ closures والـ iterators أنواعًا لا يعرفها إلا المترجم أو أنواعًا طويلة جدًا في تحديدها. يتيح لك الـ `impl Trait` syntax تحديد أن الدالة تعيد نوعًا ما ينفذ الـ `Iterator` trait بإيجاز دون الحاجة إلى كتابة نوع طويل جدًا.

ومع ذلك، يمكنك فقط استخدام `impl Trait` إذا كنت تعيد نوعًا واحدًا فقط. على سبيل المثال، هذا الكود الذي يعيد إما `NewsArticle` أو `SocialPost` مع تحديد نوع الإرجاع كـ `impl Summary` لن يعمل:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-06-impl-trait-returns-one-type/src/lib.rs:here}}
```

إرجاع إما `NewsArticle` أو `SocialPost` غير مسموح به بسبب قيود حول كيفية تنفيذ الـ `impl Trait` syntax في المترجم. سنغطي كيفية كتابة دالة بهذا السلوك في قسم ["استخدام كائنات السمات للتجريد فوق السلوك المشترك"][trait-objects]<!-- ignore --> من الفصل 18.

### استخدام قيود السمات لتنفيذ الدوال بشكل مشروط (Using Trait Bounds to Conditionally Implement Methods)

باستخدام trait bound مع كتلة `impl` تستخدم معاملات أنواع عامة، يمكننا تنفيذ الـ methods بشكل مشروط للأنواع التي تنفذ الـ traits المحددة. على سبيل المثال، النوع `Pair<T>` في القائمة 10-15 ينفذ دائمًا دالة `new` لإعادة instance جديد من `Pair<T>` (تذكر من قسم ["بناء جملة الدالة"][methods]<!-- ignore --> في الفصل 5 أن `Self` هو اسم مستعار لنوع كتلة `impl` وهو في هذه الحالة `Pair<T>`). ولكن في كتلة `impl` التالية، ينفذ `Pair<T>` دالة `cmp_display` فقط إذا كان نوعه الداخلي `T` ينفذ الـ `PartialOrd` trait الذي يتيح المقارنة _و_ الـ `Display` trait الذي يتيح الطباعة.

<Listing number="10-15" file-name="src/lib.rs" caption="تنفيذ الدوال بشكل مشروط على نوع عام اعتمادًا على قيود السمات">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-15/src/lib.rs}}
```

</Listing>

يمكننا أيضًا تنفيذ trait بشكل مشروط لأي نوع ينفذ trait آخر. تسمى تنفيذات trait على أي نوع يستوفي الـ trait bounds بـ التنفيذات الشاملة (blanket implementations) وتستخدم على نطاق واسع في مكتبة Rust القياسية. على سبيل المثال، تنفذ المكتبة القياسية الـ `ToString` trait على أي نوع ينفذ الـ `Display` trait. تبدو كتلة `impl` في المكتبة القياسية مشابهة لهذا الكود:

```rust,ignore
impl<T: Display> ToString for T {
    // --snip--
}
```

نظرًا لأن المكتبة القياسية تمتلك هذا الـ blanket implementation، يمكننا استدعاء دالة `to_string` المعرفة بواسطة الـ `ToString` trait على أي نوع ينفذ الـ `Display` trait. على سبيل المثال، يمكننا تحويل الأعداد الصحيحة (integers) إلى قيم `String` المقابلة لها بهذا الشكل لأن الأعداد الصحيحة تنفذ `Display`:

```rust
let s = 3.to_string();
```

تظهر الـ blanket implementations في التوثيق الخاص بالـ trait في قسم "المنفذون" (Implementors).

تتيح لنا الـ traits والـ trait bounds كتابة كود يستخدم معاملات أنواع عامة لتقليل التكرار ولكنها تحدد أيضًا للمترجم أننا نريد أن يمتلك النوع العام سلوكًا معينًا. يمكن للمترجم بعد ذلك استخدام معلومات الـ trait bound للتحقق من أن جميع الأنواع الملموسة المستخدمة مع كودنا توفر السلوك الصحيح. في اللغات ذات الأنواع الديناميكية (dynamically typed languages)، سنحصل على خطأ في وقت التشغيل (runtime) إذا استدعينا دالة على نوع لم يحدد تلك الدالة. لكن Rust ينقل هذه الأخطاء إلى وقت الترجمة (compile time) بحيث نضطر إلى إصلاح المشكلات قبل أن يتمكن كودنا من العمل. بالإضافة إلى ذلك، لا يتعين علينا كتابة كود يتحقق من السلوك في وقت التشغيل، لأننا تحققنا بالفعل في وقت الترجمة. القيام بذلك يحسن الأداء دون الحاجة إلى التخلي عن مرونة الأنواع العامة (generics).

[trait-objects]: ch18-02-trait-objects.html#using-trait-objects-to-abstract-over-shared-behavior
[methods]: ch05-03-method-syntax.html#method-syntax
