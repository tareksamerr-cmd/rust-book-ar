## التحكم في التدفق المختصر باستخدام `if let` و `let...else`

تتيح لك صياغة `if let` دمج `if` و `let` في طريقة أقل إسهابًا للتعامل مع القيم التي تطابق نمطًا واحدًا (pattern) مع تجاهل الباقي. لننظر إلى البرنامج في القائمة 6-6 الذي يطابق قيمة `Option<u8>` في المتغير `config_max` ولكنه يريد فقط تنفيذ الكود إذا كانت القيمة هي البديل (variant) `Some`.

<Listing number="6-6" caption="مطابقة (match) تهتم فقط بتنفيذ الكود عندما تكون القيمة `Some`">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-06/src/main.rs:here}}
```

</Listing>

إذا كانت القيمة هي `Some`، فإننا نطبع القيمة في `variant` `Some` عن طريق ربط (binding) القيمة بالمتغير `max` في `pattern`. لا نريد أن نفعل أي شيء بقيمة `None`. لإرضاء تعبير المطابقة (`match expression`)، يجب علينا إضافة `_ => ()` بعد معالجة `variant` واحد فقط، وهو كود نمطي مزعج (annoying boilerplate code) للإضافة.

بدلاً من ذلك، يمكننا كتابة هذا بطريقة أقصر باستخدام `if let`. يتصرف الكود التالي بنفس طريقة `match` في القائمة 6-6:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-12-if-let/src/main.rs:here}}
```

تأخذ صياغة `if let` نمطًا (pattern) وتعبيرًا (expression) مفصولين بعلامة يساوي. يعمل بنفس طريقة `match`، حيث يتم إعطاء `expression` إلى `match` و `pattern` هو ذراعه الأول (first arm). في هذه الحالة، `pattern` هو `Some(max)`، ويرتبط `max` بالقيمة داخل `Some`. يمكننا بعد ذلك استخدام `max` في جسم كتلة `if let` (if let block) بنفس الطريقة التي استخدمنا بها `max` في ذراع `match` المقابل. يتم تشغيل الكود في `if let block` فقط إذا كانت القيمة تطابق `pattern`.

استخدام `if let` يعني كتابة أقل، مسافة بادئة (indentation) أقل، و `boilerplate code` أقل. ومع ذلك، فإنك تفقد التحقق الشامل (exhaustive checking) الذي يفرضه `match` والذي يضمن أنك لا تنسى التعامل مع أي حالات. يعتمد الاختيار بين `match` و `if let` على ما تفعله في حالتك الخاصة وما إذا كانت اكتساب الإيجاز (conciseness) مقايضة مناسبة لفقدان `exhaustive checking`.

بمعنى آخر، يمكنك التفكير في `if let` على أنه سكر صياغي (syntax sugar) لـ `match` يقوم بتشغيل الكود عندما تطابق القيمة `pattern` واحدًا ثم تتجاهل جميع القيم الأخرى.

يمكننا تضمين `else` مع `if let`. كتلة الكود التي تأتي مع `else` هي نفسها كتلة الكود التي ستأتي مع حالة `_` في `match expression` المكافئ لـ `if let` و `else`. تذكر تعريف التعداد (enum) `Coin` في القائمة 6-4، حيث احتوى البديل `Quarter` أيضًا على قيمة `UsState`. إذا أردنا عد جميع العملات غير الربعية التي نراها مع الإعلان أيضًا عن حالة الأرباع، يمكننا القيام بذلك باستخدام `match expression`، مثل هذا:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-13-count-and-announce-match/src/main.rs:here}}
```

أو يمكننا استخدام تعبير `if let` و `else`، مثل هذا:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-14-count-and-announce-if-let-else/src/main.rs:here}}
```

## البقاء على "المسار السعيد" (Happy Path) باستخدام `let...else`

النمط الشائع هو إجراء بعض العمليات الحسابية عندما تكون القيمة موجودة وإرجاع قيمة افتراضية (default value) بخلاف ذلك. استمرارًا في مثالنا للعملات المعدنية بقيمة `UsState`، إذا أردنا أن نقول شيئًا مضحكًا اعتمادًا على عمر الولاية على الربع، فقد نقدم طريقة (method) على `UsState` للتحقق من عمر الولاية، مثل هذا:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-07/src/main.rs:state}}
```

بعد ذلك، قد نستخدم `if let` للمطابقة على نوع العملة، مع تقديم متغير `state` داخل جسم الشرط (condition)، كما في القائمة 6-7.

<Listing number="6-7" caption="التحقق مما إذا كانت الولاية موجودة في عام 1900 باستخدام شروط متداخلة داخل `if let`">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-07/src/main.rs:describe}}
```

</Listing>

هذا ينجز المهمة، ولكنه دفع العمل إلى جسم عبارة `if let`، وإذا كان العمل الذي يتعين القيام به أكثر تعقيدًا، فقد يكون من الصعب متابعة كيفية ارتباط الفروع (branches) ذات المستوى الأعلى بالضبط. يمكننا أيضًا الاستفادة من حقيقة أن التعبيرات (expressions) تنتج قيمة إما لإنتاج `state` من `if let` أو للعودة مبكرًا (return early)، كما في القائمة 6-8. (يمكنك القيام بشيء مشابه باستخدام `match` أيضًا.)

<Listing number="6-8" caption="استخدام `if let` لإنتاج قيمة أو العودة مبكرًا">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-08/src/main.rs:describe}}
```

</Listing>

هذا مزعج بعض الشيء للمتابعة بطريقته الخاصة، على الرغم من ذلك! ينتج أحد فروع `if let` قيمة، والآخر يعود من الدالة (function) بالكامل.

لجعل هذا النمط الشائع أسهل في التعبير، تحتوي Rust على `let...else`. تأخذ صياغة `let...else` نمطًا (pattern) على الجانب الأيسر وتعبيرًا (expression) على الجانب الأيمن، مشابهًا جدًا لـ `if let`، ولكن ليس لديها فرع `if`، بل فرع `else` فقط. إذا طابق `pattern`، فسيربط القيمة من `pattern` في النطاق الخارجي (outer scope). إذا لم يطابق `pattern`، فسيتدفق البرنامج إلى ذراع `else`، والذي يجب أن يعود من `function`.

في القائمة 6-9، يمكنك أن ترى كيف تبدو القائمة 6-8 عند استخدام `let...else` بدلاً من `if let`.

<Listing number="6-9" caption="استخدام `let...else` لتوضيح التدفق عبر الدالة">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-09/src/main.rs:describe}}
```

</Listing>

لاحظ أنه يبقى على "المسار السعيد" (happy path) في الجسم الرئيسي لـ `function` بهذه الطريقة، دون أن يكون لديه تدفق تحكم (control flow) مختلف بشكل كبير لفرعين بالطريقة التي فعلها `if let`.

إذا كان لديك موقف يحتوي فيه برنامجك على منطق (logic) مطول جدًا للتعبير عنه باستخدام `match`، فتذكر أن `if let` و `let...else` موجودان في صندوق أدوات Rust الخاص بك أيضًا.

## ملخص (Summary)

لقد غطينا الآن كيفية استخدام التعدادات (enums) لإنشاء أنواع مخصصة (custom types) يمكن أن تكون واحدة من مجموعة من القيم المعددة. لقد أوضحنا كيف يساعدك نوع `Option<T>` في المكتبة القياسية (standard library) على استخدام نظام الأنواع (type system) لمنع الأخطاء. عندما تحتوي قيم `enum` على بيانات بداخلها، يمكنك استخدام `match` أو `if let` لاستخراج واستخدام تلك القيم، اعتمادًا على عدد الحالات التي تحتاج إلى التعامل معها.

يمكن لبرامج Rust الخاصة بك الآن التعبير عن المفاهيم في مجالك باستخدام الهياكل (structs) و `enums`. يضمن إنشاء `custom types` لاستخدامها في واجهة برمجة التطبيقات (API) الخاصة بك سلامة الأنواع (type safety): سيضمن المترجم (compiler) أن الدوال (functions) الخاصة بك تحصل فقط على قيم من النوع الذي تتوقعه كل دالة.

من أجل توفير `API` جيد التنظيم لمستخدميك يكون سهل الاستخدام ولا يكشف إلا عما سيحتاجه المستخدمون بالضبط، دعنا ننتقل الآن إلى وحدات Rust (Rust’s modules).
