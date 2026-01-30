<!-- Old headings. Do not remove or links may break. -->

<a id="defining-modules-to-control-scope-and-privacy"></a>

## التحكم في النطاق والخصوصية باستخدام الوحدات البرمجية

في هذا القسم، سنتحدث عن الوحدات البرمجية (modules) وأجزاء أخرى من نظام الوحدات البرمجية (module system)، وتحديداً المسارات (paths)، التي تسمح لك بتسمية العناصر؛ والكلمة المفتاحية `use` التي تجلب المسار (path) إلى النطاق (scope)؛ والكلمة المفتاحية `pub` لجعل العناصر عامة (public). سنناقش أيضاً الكلمة المفتاحية `as` والطرود الخارجية (external packages) وعامل الشمول (glob operator).

### ورقة غش الوحدات البرمجية

قبل أن ننتقل إلى تفاصيل modules و paths، نقدم هنا مرجعاً سريعاً حول كيفية عمل modules و paths والكلمة المفتاحية `use` والكلمة المفتاحية `pub` في المترجم (compiler)، وكيف ينظم معظم المطورين كودهم. سنمر بأمثلة على كل من هذه القواعد طوال هذا الفصل، ولكن هذا مكان رائع للرجوع إليه كتذكير بكيفية عمل modules.

- **البدء من جذر الكريت (crate root)**: عند تصريف (compiling) كريت (crate)، يبحث compiler أولاً في ملف crate root (عادةً ما يكون _src/lib.rs_ لكريت المكتبة و _src/main.rs_ لكريت ثنائي) عن كود لتصريفه.
- **التصريح عن الوحدات البرمجية (Declaring modules)**: في ملف crate root، يمكنك التصريح عن modules جديدة؛ لنفترض أنك صرحت عن وحدة "garden" باستخدام `mod garden;`. سيبحث compiler عن كود الوحدة في هذه الأماكن:
  - مضمناً (Inline)، داخل أقواس متعرجة تحل محل الفاصلة المنقوطة التي تلي `mod garden`
  - في الملف _src/garden.rs_
  - في الملف _src/garden/mod.rs_
- **التصريح عن الوحدات الفرعية (Declaring submodules)**: في أي ملف آخر غير crate root، يمكنك التصريح عن وحدات فرعية (submodules). على سبيل المثال، قد تصرح عن `mod vegetables;` في _src/garden.rs_. سيبحث compiler عن كود submodule داخل المجلد المسمى باسم الوحدة الأب (parent module) في هذه الأماكن:
  - مضمناً، مباشرة بعد `mod vegetables` داخل أقواس متعرجة بدلاً من الفاصلة المنقوطة
  - في الملف _src/garden/vegetables.rs_
  - في الملف _src/garden/vegetables/mod.rs_
- **المسارات إلى الكود في الوحدات البرمجية**: بمجرد أن تصبح module جزءاً من crate الخاص بك، يمكنك الرجوع إلى الكود في تلك module من أي مكان آخر في نفس crate، طالما تسمح قواعد الخصوصية (privacy rules) بذلك، باستخدام path المؤدي إلى الكود. على سبيل المثال، سيتم العثور على النوع `Asparagus` في وحدة vegetables الخاصة بـ garden في `crate::garden::vegetables::Asparagus`.
- **خاص مقابل عام (Private vs. public)**: الكود داخل module يكون خاصاً (private) عن وحداته الأب بشكل افتراضي. لجعل module عامة (public)، صرح عنها باستخدام `pub mod` بدلاً من `mod`. لجعل العناصر داخل public module عامة أيضاً، استخدم `pub` قبل التصريح عنها.
- **الكلمة المفتاحية `use`**: داخل scope، تنشئ الكلمة المفتاحية `use` اختصارات للعناصر لتقليل تكرار paths الطويلة. في أي scope يمكنه الرجوع إلى `crate::garden::vegetables::Asparagus` يمكنك إنشاء اختصار باستخدام `use crate::garden::vegetables::Asparagus;` ومنذ ذلك الحين فصاعداً ستحتاج فقط إلى كتابة `Asparagus` لاستخدام ذلك النوع في scope.

هنا، ننشئ binary crate باسم `backyard` يوضح هذه القواعد. يحتوي مجلد crate، المسمى أيضاً _backyard_، على هذه الملفات والمجلدات:

```text
backyard
├── Cargo.lock
├── Cargo.toml
└── src
    ├── garden
    │   └── vegetables.rs
    ├── garden.rs
    └── main.rs
```

ملف crate root في هذه الحالة هو _src/main.rs_، ويحتوي على:

<Listing file-name="src/main.rs">

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/main.rs}}
```

</Listing>

يخبر سطر `pub mod garden;` المترجم بتضمين الكود الذي يجده في _src/garden.rs_، وهو:

<Listing file-name="src/garden.rs">

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/garden.rs}}
```

</Listing>

هنا، تعني `pub mod vegetables;` أن الكود في _src/garden/vegetables.rs_ مضمن أيضاً. هذا الكود هو:

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/garden/vegetables.rs}}
```

الآن دعونا ندخل في تفاصيل هذه القواعد ونستعرضها عملياً!

### تجميع الكود ذو الصلة في وحدات برمجية

تسمح لنا _Modules_ بتنظيم الكود داخل crate لسهولة القراءة وإعادة الاستخدام. تسمح لنا modules أيضاً بالتحكم في _خصوصية_ العناصر لأن الكود داخل module يكون private بشكل افتراضي. العناصر الخاصة هي تفاصيل تنفيذ داخلية (internal implementation details) غير متاحة للاستخدام الخارجي. يمكننا اختيار جعل modules والعناصر داخلها public، مما يكشفها للسماح للكود الخارجي باستخدامها والاعتماد عليها.

كمثال، لنكتب library crate يوفر وظائف مطعم. سنحدد تواقيع الدوال (function signatures) ولكن سنترك أجسامها فارغة للتركيز على تنظيم الكود بدلاً من تنفيذ (implementation) المطعم.

في صناعة المطاعم، يشار إلى بعض أجزاء المطعم باسم "واجهة المطعم" (front of house) وأجزاء أخرى باسم "خلفية المطعم" (back of house). _Front of house_ هو المكان الذي يتواجد فيه الزبائن؛ وهذا يشمل المكان الذي يجلس فيه المضيفون الزبائن، ويأخذ فيه النادلون الطلبات والمدفوعات، ويقوم فيه السقاة بإعداد المشروبات. _Back of house_ هو المكان الذي يعمل فيه الطهاة في المطبخ، ويقوم فيه غاسلو الأطباق بالتنظيف، ويقوم فيه المديرون بالعمل الإداري.

لهيكلة crate الخاص بنا بهذه الطريقة، يمكننا تنظيم دواله في modules متداخلة. أنشئ مكتبة جديدة باسم `restaurant` عن طريق تشغيل `cargo new restaurant --lib`. ثم أدخل الكود الموجود في القائمة 7-1 في _src/lib.rs_ لتعريف بعض modules وتواقيع الدوال؛ هذا الكود هو قسم front of house.

<Listing number="7-1" file-name="src/lib.rs" caption="وحدة `front_of_house` تحتوي على وحدات أخرى تحتوي بدورها على دوال">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-01/src/lib.rs}}
```

</Listing>

نحدد module باستخدام الكلمة المفتاحية `mod` متبوعة باسم module (في هذه الحالة، `front_of_house`). ثم يوضع جسم module داخل أقواس متعرجة. داخل modules، يمكننا وضع modules أخرى، كما في هذه الحالة مع الوحدتين `hosting` و `serving`. يمكن أن تحتوي modules أيضاً على تعريفات لعناصر أخرى، مثل الهياكل (structs) والتعدادات (enums) والثوابت (constants) والسمات (traits) وكما في القائمة 7-1، الدوال (functions).

باستخدام modules، يمكننا تجميع التعريفات ذات الصلة معاً وتسمية سبب ارتباطها. يمكن للمبرمجين الذين يستخدمون هذا الكود التنقل فيه بناءً على المجموعات بدلاً من الاضطرار إلى قراءة جميع التعريفات، مما يسهل العثور على التعريفات ذات الصلة بهم. سيعرف المبرمجون الذين يضيفون وظائف جديدة إلى هذا الكود مكان وضع الكود للحفاظ على تنظيم البرنامج.

ذكرنا سابقاً أن _src/main.rs_ و _src/lib.rs_ يسمى كل منهما _crate roots_. والسبب في تسميتهما هو أن محتويات أي من هذين الملفين تشكل module تسمى `crate` في جذر هيكل الوحدات البرمجية للكريت، والمعروف باسم شجرة الوحدات البرمجية (module tree).

توضح القائمة 7-2 module tree للهيكل الموجود في القائمة 7-1.

<Listing number="7-2" caption="شجرة الوحدات البرمجية (module tree) للكود في القائمة 7-1">

```text
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```

</Listing>

توضح هذه الشجرة كيف تتداخل بعض modules داخل modules أخرى؛ على سبيل المثال، `hosting` يتداخل داخل `front_of_house`. توضح الشجرة أيضاً أن بعض modules هي أشقاء (siblings)، مما يعني أنها معرفة في نفس module؛ `hosting` و `serving` هما siblings معرفان داخل `front_of_house`. إذا كانت الوحدة A محتواة داخل الوحدة B، فإننا نقول إن الوحدة A هي الابن (child) للوحدة B وأن الوحدة B هي الأب (parent) للوحدة A. لاحظ أن module tree بالكامل متجذرة تحت module الضمنية المسماة `crate`.

قد تذكرك module tree بشجرة مجلدات نظام الملفات على جهاز الكمبيوتر الخاص بك؛ هذا تشبيه دقيق للغاية! تماماً مثل المجلدات في نظام الملفات، تستخدم modules لتنظيم كودك. وتماماً مثل الملفات في المجلد، نحتاج إلى طريقة للعثور على modules الخاصة بنا.
