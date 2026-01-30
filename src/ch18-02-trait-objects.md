<!-- Old headings. Do not remove or links may break. -->

<a id="using-trait-objects-that-allow-for-values-of-different-types"></a>

## استخدام كائنات السمات للتجريد فوق السلوك المشترك (Using Trait Objects to Abstract over Shared Behavior)

في الفصل 8، ذكرنا أن أحد قيود المتجهات (vectors) هو أنها يمكن أن تخزن عناصر من نوع واحد فقط. لقد أنشأنا حلاً بديلاً في القائمة 8-9 حيث عرفنا تعداد `SpreadsheetCell` (enum) يحتوي على متغيرات (variants) لحمل الأعداد الصحيحة، والأعداد العشرية، والنصوص. كان هذا يعني أنه يمكننا تخزين أنواع مختلفة من البيانات في كل خلية مع استمرار وجود vector يمثل صفًا من الخلايا. هذا حل ممتاز عندما تكون العناصر القابلة للتبديل لدينا هي مجموعة ثابتة من الأنواع التي نعرفها عند ترجمة الكود الخاص بنا.

ومع ذلك، في بعض الأحيان نريد لمستخدم المكتبة الخاصة بنا أن يكون قادرًا على توسيع مجموعة الأنواع الصالحة في موقف معين. لتوضيح كيف يمكننا تحقيق ذلك، سننشئ مثالاً لأداة واجهة مستخدم رسومية (GUI) تمر عبر قائمة من العناصر، وتستدعي دالة `draw` على كل منها لرسمها على الشاشة - وهو أسلوب شائع لأدوات الـ GUI. سننشئ حزمة مكتبة (library crate) تسمى `gui` تحتوي على هيكلية مكتبة GUI. قد تتضمن هذه الـ crate بعض الأنواع ليستخدمها الأشخاص، مثل `Button` أو `TextField`. بالإضافة إلى ذلك، سيرغب مستخدمو `gui` في إنشاء أنواعهم الخاصة التي يمكن رسمها: على سبيل المثال، قد يضيف أحد المبرمجين `Image` وقد يضيف مبرمج آخر `SelectBox`.

في وقت كتابة المكتبة، لا يمكننا معرفة وتعريف جميع الأنواع التي قد يرغب المبرمجون الآخرون في إنشائها. لكننا نعلم أن `gui` بحاجة إلى تتبع العديد من القيم ذات الأنواع المختلفة، وهي بحاجة إلى استدعاء دالة `draw` على كل من هذه القيم ذات الأنواع المختلفة. لا تحتاج المكتبة إلى معرفة ما سيحدث بالضبط عندما نستدعي دالة `draw` بل تحتاج فقط إلى معرفة أن القيمة ستمتلك تلك الدالة متاحة لنا لاستدعائها.

للقيام بذلك في لغة تدعم الوراثة (inheritance)، قد نعرف فئة (class) تسمى `Component` تمتلك دالة تسمى `draw`. الفئات الأخرى، مثل `Button` و `Image` و `SelectBox` سترث من `Component` وبالتالي ترث دالة `draw`. يمكن لكل منها تجاوز (override) دالة `draw` لتعريف سلوكها المخصص، ولكن يمكن للإطار البرمجي (framework) التعامل مع جميع الأنواع كما لو كانت مثيلات (instances) من `Component` واستدعاء `draw` عليها. ولكن نظرًا لأن Rust لا تمتلك inheritance، فنحن بحاجة إلى طريقة أخرى لهيكلة مكتبة `gui` للسماح للمستخدمين بإنشاء أنواع جديدة متوافقة مع المكتبة.

### تعريف سمة للسلوك المشترك (Defining a Trait for Common Behavior)

لتنفيذ السلوك الذي نريده لـ `gui` سنعرف سمة (trait) تسمى `Draw` ستمتلك دالة واحدة تسمى `draw`. بعد ذلك، يمكننا تعريف vector يأخذ كائن سمة (trait object). يشير الـ *trait object* إلى كل من مثيل لنوع ينفذ الـ trait المحدد وجدول يستخدم للبحث عن دوال (methods) الـ trait على ذلك النوع في وقت التشغيل (runtime). ننشئ trait object عن طريق تحديد نوع من المؤشرات (pointers)، مثل مرجع (reference) أو مؤشر ذكي من نوع `Box<T>` ثم الكلمة المفتاحية `dyn` ثم تحديد الـ trait ذي الصلة. (سنتحدث عن سبب وجوب استخدام trait objects لمؤشر في قسم ["الأنواع ذات الحجم الديناميكي وسمة `Sized`"][dynamically-sized]<!-- ignore --> في الفصل 20). يمكننا استخدام trait objects بدلاً من نوع عام (generic) أو نوع ملموس (concrete type). أينما نستخدم trait object، سيضمن نظام الأنواع في Rust في وقت الترجمة (compile time) أن أي قيمة مستخدمة في ذلك السياق ستنفذ الـ trait الخاص بـ trait object. وبالتالي، لا نحتاج إلى معرفة جميع الأنواع الممكنة في وقت الترجمة.

لقد ذكرنا أننا في Rust نمتنع عن تسمية الهياكل (structs) والتعدادات (enums) بـ "كائنات" (objects) لتمييزها عن كائنات اللغات الأخرى. في الـ struct أو الـ enum، تكون البيانات في حقول الـ struct والسلوك في كتل `impl` منفصلة، بينما في اللغات الأخرى، غالبًا ما يطلق على البيانات والسلوك المدمجين في مفهوم واحد اسم كائن. تختلف الـ trait objects عن الكائنات في اللغات الأخرى في أننا لا نستطيع إضافة بيانات إلى trait object. الـ trait objects ليست مفيدة بشكل عام مثل الكائنات في اللغات الأخرى: غرضها المحدد هو السماح بالتجريد (abstraction) عبر السلوك المشترك.

توضح القائمة 18-3 كيفية تعريف trait يسمى `Draw` مع دالة واحدة تسمى `draw`.

<Listing number="18-3" file-name="src/lib.rs" caption="تعريف سمة `Draw` ">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-03/src/lib.rs}}
```

</Listing>

يجب أن يبدو بناء الجملة (syntax) هذا مألوفًا من نقاشاتنا حول كيفية تعريف الـ traits في الفصل 10. بعد ذلك يأتي بعض الـ syntax الجديد: توضح القائمة 18-4 تعريف struct يسمى `Screen` يحمل vector يسمى `components`. هذا الـ vector هو من نوع `Box<dyn Draw>` وهو trait object؛ إنه بديل لأي نوع داخل `Box` ينفذ الـ `Draw` trait.

<Listing number="18-4" file-name="src/lib.rs" caption="تعريف هيكل `Screen` مع حقل `components` يحمل متجهاً من كائنات السمات التي تنفذ سمة `Draw` ">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-04/src/lib.rs:here}}
```

</Listing>

على هيكل `Screen` سنعرف دالة تسمى `run` ستستدعي دالة `draw` على كل من الـ `components` الخاصة بها، كما هو موضح في القائمة 18-5.

<Listing number="18-5" file-name="src/lib.rs" caption="دالة `run` على `Screen` تستدعي دالة `draw` على كل مكون">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-05/src/lib.rs:here}}
```

</Listing>

يعمل هذا بشكل مختلف عن تعريف struct يستخدم معامل نوع عام (generic type parameter) مع قيود السمات (trait bounds). يمكن استبدال generic type parameter بنوع ملموس واحد فقط في كل مرة، بينما تسمح الـ trait objects لعدة أنواع ملموسة بملء مكان الـ trait object في وقت التشغيل. على سبيل المثال، كان بإمكاننا تعريف هيكل `Screen` باستخدام نوع عام و trait bound، كما في القائمة 18-6.

<Listing number="18-6" file-name="src/lib.rs" caption="تنفيذ بديل لهيكل `Screen` ودالته `run` باستخدام الأنواع العامة وقيود السمات">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-06/src/lib.rs:here}}
```

</Listing>

هذا يقيدنا بمثيل من `Screen` يمتلك قائمة من المكونات كلها من نوع `Button` أو كلها من نوع `TextField`. إذا كنت ستمتلك دائمًا مجموعات متجانسة (homogeneous collections)، فإن استخدام الـ generics والـ trait bounds يفضل لأن التعريفات سيتم توحيد شكلها (monomorphized) في وقت الترجمة لاستخدام الأنواع الملموسة.

من ناحية أخرى، مع الدالة التي تستخدم trait objects، يمكن لمثيل واحد من `Screen` أن يحمل `Vec<T>` يحتوي على `Box<Button>` بالإضافة إلى `Box<TextField>`. دعنا ننظر في كيفية عمل ذلك، ثم سنتحدث عن الآثار المترتبة على أداء وقت التشغيل.

### تنفيذ السمة (Implementing the Trait)

الآن سنضيف بعض الأنواع التي تنفذ الـ `Draw` trait. سنوفر نوع `Button`. مرة أخرى، تنفيذ مكتبة GUI فعليًا هو خارج نطاق هذا الكتاب، لذا لن تمتلك دالة `draw` أي تنفيذ مفيد في جسمها. لتخيل كيف قد يبدو التنفيذ، قد يمتلك هيكل `Button` حقولاً لـ `width` و `height` و `label` كما هو موضح في القائمة 18-7.

<Listing number="18-7" file-name="src/lib.rs" caption="هيكل `Button` ينفذ سمة `Draw` ">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-07/src/lib.rs:here}}
```

</Listing>

ستختلف حقول `width` و `height` و `label` في `Button` عن الحقول في المكونات الأخرى؛ على سبيل المثال، قد يمتلك نوع `TextField` نفس تلك الحقول بالإضافة إلى حقل `placeholder`. كل نوع من الأنواع التي نريد رسمها على الشاشة سينفذ الـ `Draw` trait ولكنه سيستخدم كودًا مختلفًا في دالة `draw` لتعريف كيفية رسم ذلك النوع المعين، كما فعل `Button` هنا (بدون كود الـ GUI الفعلي، كما ذكرنا). قد يمتلك نوع `Button` على سبيل المثال كتلة `impl` إضافية تحتوي على methods متعلقة بما يحدث عندما ينقر المستخدم على الزر. هذه الأنواع من الـ methods لن تنطبق على أنواع مثل `TextField`.

إذا قرر شخص يستخدم مكتبتنا تنفيذ هيكل `SelectBox` يمتلك حقول `width` و `height` و `options` فإنه سينفذ الـ `Draw` trait على نوع `SelectBox` أيضًا، كما هو موضح في القائمة 18-8.

<Listing number="18-8" file-name="src/main.rs" caption="حزمة أخرى تستخدم `gui` وتنفذ سمة `Draw` على هيكل `SelectBox` ">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-08/src/main.rs:here}}
```

</Listing>

يمكن لمستخدم مكتبتنا الآن كتابة دالة `main` الخاصة به لإنشاء مثيل من `Screen`. إلى مثيل `Screen` يمكنه إضافة `SelectBox` و `Button` عن طريق وضع كل منهما في `Box<T>` ليصبح trait object. يمكنه بعد ذلك استدعاء دالة `run` على مثيل `Screen` والتي ستستدعي `draw` على كل من المكونات. توضح القائمة 18-9 هذا التنفيذ.

<Listing number="18-9" file-name="src/main.rs" caption="استخدام كائنات السمات لتخزين قيم من أنواع مختلفة تنفذ نفس السمة">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-09/src/main.rs:here}}
```

</Listing>

عندما كتبنا المكتبة، لم نكن نعلم أن شخصًا ما قد يضيف نوع `SelectBox` ولكن تنفيذنا لـ `Screen` كان قادرًا على العمل على النوع الجديد ورسمه لأن `SelectBox` ينفذ الـ `Draw` trait، مما يعني أنه ينفذ دالة `draw`.

هذا المفهوم - المتمثل في الاهتمام فقط بالرسائل التي تستجيب لها القيمة بدلاً من النوع الملموس للقيمة - يشبه مفهوم *كتابة البطة* (duck typing) في اللغات ذات الأنواع الديناميكية: إذا كان يمشي مثل البطة ويصيح مثل البطة، فلا بد أنه بطة! في تنفيذ `run` على `Screen` في القائمة 18-5، لا تحتاج `run` إلى معرفة النوع الملموس لكل مكون. إنها لا تتحقق مما إذا كان المكون مثيلاً لـ `Button` أو `SelectBox` بل تستدعي فقط دالة `draw` على المكون. من خلال تحديد `Box<dyn Draw>` كنوع للقيم في vector الـ `components` فقد عرفنا `Screen` بحيث يحتاج إلى قيم يمكننا استدعاء دالة `draw` عليها.

ميزة استخدام trait objects ونظام الأنواع في Rust لكتابة كود مشابه للكود الذي يستخدم duck typing هي أننا لا نضطر أبدًا للتحقق مما إذا كانت القيمة تنفذ دالة معينة في وقت التشغيل أو القلق بشأن الحصول على أخطاء إذا كانت القيمة لا تنفذ دالة ولكننا استدعيناها على أي حال. لن يترجم Rust الكود الخاص بنا إذا كانت القيم لا تنفذ الـ traits التي تحتاجها الـ trait objects.

على سبيل المثال، توضح القائمة 18-10 ما يحدث إذا حاولنا إنشاء `Screen` مع `String` كمكون.

<Listing number="18-10" file-name="src/main.rs" caption="محاولة استخدام نوع لا ينفذ سمة كائن السمة">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-10/src/main.rs}}
```

</Listing>

سنحصل على هذا الخطأ لأن `String` لا ينفذ الـ `Draw` trait:

```console
{{#include ../listings/ch18-oop/listing-18-10/output.txt}}
```

يخبرنا هذا الخطأ أننا إما نمرر شيئًا إلى `Screen` لم نكن نقصد تمريره وبالتالي يجب تمرير نوع مختلف، أو يجب علينا تنفيذ `Draw` على `String` بحيث يتمكن `Screen` من استدعاء `draw` عليه.

<!-- Old headings. Do not remove or links may break. -->

<a id="trait-objects-perform-dynamic-dispatch"></a>

### إجراء الإرسال الديناميكي (Performing Dynamic Dispatch)

تذكر في قسم ["أداء الكود الذي يستخدم الأنواع العامة"][performance-of-code-using-generics]<!-- ignore --> في الفصل 10 نقاشنا حول عملية الـ monomorphization التي يجريها المترجم على الـ generics: ينشئ المترجم تنفيذات غير عامة للدوال والـ methods لكل نوع ملموس نستخدمه بدلاً من generic type parameter. الكود الناتج عن monomorphization يقوم بـ إرسال ثابت (static dispatch)، وهو عندما يعرف المترجم الدالة التي تستدعيها في وقت الترجمة. هذا على عكس الإرسال الديناميكي (dynamic dispatch)، وهو عندما لا يستطيع المترجم معرفة الدالة التي تستدعيها في وقت الترجمة. في حالات الـ dynamic dispatch، يصدر المترجم كودًا سيعرف في وقت التشغيل الدالة التي يجب استدعاؤها.

عندما نستخدم trait objects، يجب على Rust استخدام dynamic dispatch. لا يعرف المترجم جميع الأنواع التي قد تستخدم مع الكود الذي يستخدم trait objects، لذا فهو لا يعرف أي دالة منفذة على أي نوع يجب استدعاؤها. بدلاً من ذلك، في وقت التشغيل، يستخدم Rust المؤشرات داخل trait object لمعرفة الدالة التي يجب استدعاؤها. هذا البحث يتطلب تكلفة في وقت التشغيل لا تحدث مع الـ static dispatch. يمنع الـ dynamic dispatch المترجم أيضًا من اختيار تضمين (inline) كود الدالة، مما يمنع بدوره بعض التحسينات (optimizations)، ولدى Rust بعض القواعد حول أين يمكنك وأين لا يمكنك استخدام dynamic dispatch، تسمى توافق dyn (dyn compatibility). تلك القواعد خارج نطاق هذا النقاش، ولكن يمكنك قراءة المزيد عنها [في المرجع][dyn-compatibility]<!-- ignore -->. ومع ذلك، فقد حصلنا على مرونة إضافية في الكود الذي كتبناه في القائمة 18-5 وتمكنا من دعمه في القائمة 18-9، لذا فهي مقايضة يجب أخذها في الاعتبار.

[performance-of-code-using-generics]: ch10-01-syntax.html#performance-of-code-using-generics
[dynamically-sized]: ch20-03-advanced-types.html#dynamically-sized-types-and-the-sized-trait
[dyn-compatibility]: https://doc.rust-lang.org/reference/items/traits.html#dyn-compatibility
