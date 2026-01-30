## تشغيل الشفرة البرمجية عند التنظيف باستخدام سمة `Drop` (Drop Trait)

السمة (Trait) الثانية المهمة لنمط المؤشر الذكي (Smart Pointer) هي `Drop` التي تسمح لك بتخصيص ما يحدث عندما يوشك متغير على الخروج من النطاق (Scope). يمكنك تقديم تنفيذ (Implementation) لـ `Drop` Trait على أي نوع (Type)، ويمكن استخدام تلك الشفرة البرمجية (Code) لتحرير الموارد (Resources) مثل الملفات أو اتصالات الشبكة.

نحن نقدم `Drop` في سياق Smart Pointers لأن وظائف `Drop` Trait تُستخدم دائماً تقريباً عند تنفيذ Smart Pointer. على سبيل المثال، عندما يتم إسقاط (Drop) ‏`Box<T>` فإنه سيقوم بإلغاء تخصيص (Deallocate) المساحة في الذاكرة الكومة (Heap) التي يشير إليها الصندوق.

في بعض اللغات، وبالنسبة لبعض الأنواع، يجب على المبرمج استدعاء Code لتحرير الذاكرة أو Resources في كل مرة ينتهي فيها من استخدام مثيل (Instance) من تلك الأنواع. تشمل الأمثلة مقابض الملفات (File Handles) والمآخذ (Sockets) والأقفال (Locks). إذا نسي المبرمج ذلك، فقد يصبح النظام محملاً بشكل زائد وينهار. في Rust، يمكنك تحديد تشغيل جزء معين من Code كلما خرجت قيمة من Scope، وسيقوم المترجم (Compiler) بإدراج هذا Code تلقائياً. ونتيجة لذلك، لا تحتاج إلى توخي الحذر بشأن وضع Code التنظيف في كل مكان ينتهي فيه استخدام Instance من Type معين—ولن تسرب الموارد (Leak Resources) أبداً!

تحدد Code المراد تشغيله عندما تخرج قيمة من Scope من خلال تنفيذ `Drop` Trait. يتطلب منك `Drop` Trait تنفيذ تابع (Method) واحد يسمى `drop` يأخذ مرجعاً قابلاً للتعديل (Mutable Reference) لـ `self`. لرؤية متى تستدعي Rust تابع `drop` لنقم بتنفيذ `drop` مع عبارات `println!` في الوقت الحالي.

تظهر القائمة 15-14 هيكلاً (Struct) باسم `CustomSmartPointer` وظيفته المخصصة الوحيدة هي أنه سيطبع `Dropping CustomSmartPointer!` عندما يخرج Instance من Scope، لإظهار متى تقوم Rust بتشغيل Method ‏`drop`.

<Listing number="15-14" file-name="src/main.rs" caption="هيكل `CustomSmartPointer` الذي ينفذ سمة `Drop` حيث سنضع شفرة التنظيف الخاصة بنا">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-14/src/main.rs}}
```

</Listing>

يتم تضمين `Drop` Trait في التمهيد (Prelude)، لذا لا نحتاج إلى جلبه إلى Scope. نقوم بتنفيذ `Drop` Trait على `CustomSmartPointer` ونوفر Implementation لـ Method ‏`drop` الذي يستدعي `println!`. جسم (Body) Method ‏`drop` هو المكان الذي تضع فيه أي منطق (Logic) تريد تشغيله عندما يخرج Instance من نوعك من Scope. نحن نطبع بعض النصوص هنا لتوضيح متى ستستدعي Rust تابع `drop` بصرياً.

في `main` ننشئ مثيلين (Instances) من `CustomSmartPointer` ثم نطبع `CustomSmartPointers created`. في نهاية `main` ستخرج Instances الخاصة بنا من `CustomSmartPointer` عن Scope، وستقوم Rust باستدعاء Code الذي وضعناه في Method ‏`drop` وطباعة رسالتنا النهائية. لاحظ أننا لم نكن بحاجة لاستدعاء Method ‏`drop` صراحة.

عند تشغيل هذا البرنامج، سنرى المخرجات (Output) التالية:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-14/output.txt}}
```

قامت Rust تلقائياً باستدعاء `drop` لنا عندما خرجت Instances الخاصة بنا من Scope، مستدعية Code الذي حددناه. يتم إسقاط المتغيرات (Variables) بترتيب عكسي لإنشائها، لذا تم إسقاط `d` قبل `c`. الغرض من هذا المثال هو إعطاؤك دليلاً مرئياً حول كيفية عمل Method ‏`drop`؛ عادةً ما تحدد Code التنظيف الذي يحتاجه نوعك للتشغيل بدلاً من رسالة طباعة.

<!-- Old headings. Do not remove or links may break. -->

<a id="dropping-a-value-early-with-std-mem-drop"></a>

لسوء الحظ، ليس من السهل تعطيل وظيفة `drop` التلقائية. لا يكون تعطيل `drop` ضرورياً عادةً؛ فالمغزى الكامل من `Drop` Trait هو أنه يتم الاعتناء به تلقائياً. ومع ذلك، قد ترغب أحياناً في تنظيف قيمة مبكراً. أحد الأمثلة هو عند استخدام Smart Pointers التي تدير Locks: قد ترغب في فرض Method ‏`drop` الذي يحرر القفل حتى يتمكن Code آخر في نفس Scope من الحصول على القفل. لا تسمح لك Rust باستدعاء Method ‏`drop` الخاص بـ `Drop` Trait يدوياً؛ بدلاً من ذلك، يتعين عليك استدعاء Function ‏`std::mem::drop` التي توفرها المكتبة القياسية (Standard Library) إذا كنت تريد فرض إسقاط قيمة قبل نهاية Scope الخاص بها.

محاولة استدعاء Method ‏`drop` الخاص بـ `Drop` Trait يدوياً عن طريق تعديل Function ‏`main` من القائمة 15-14 لن تنجح، كما هو موضح في القائمة 15-15.

<Listing number="15-15" file-name="src/main.rs" caption="محاولة استدعاء تابع `drop` من سمة `Drop` يدوياً للتنظيف المبكر">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-15/src/main.rs:here}}
```

</Listing>

عندما نحاول ترجمة (Compile) هذا Code، سنحصل على هذا الخطأ (Error):

```console
{{#include ../listings/ch15-smart-pointers/listing-15-15/output.txt}}
```

تنص رسالة Error هذه على أنه لا يُسمح لنا باستدعاء `drop` صراحة. تستخدم رسالة الخطأ مصطلح *المُهدم* (Destructor)، وهو المصطلح البرمجي العام لـ Function تقوم بتنظيف Instance. يعتبر *Destructor* مناظراً لـ *المُنشئ* (Constructor) الذي ينشئ Instance. دالة `drop` في Rust هي Destructor واحد محدد.

لا تسمح لنا Rust باستدعاء `drop` صراحة، لأن Rust ستستمر في استدعاء `drop` تلقائياً على القيمة في نهاية `main`. سيؤدي هذا إلى خطأ التحرير المزدوج (Double Free Error) لأن Rust ستحاول تنظيف نفس القيمة مرتين.

لا يمكننا تعطيل الإدراج التلقائي لـ `drop` عندما تخرج قيمة من Scope، ولا يمكننا استدعاء Method ‏`drop` صراحة. لذا، إذا كنا بحاجة إلى فرض تنظيف قيمة مبكراً، فإننا نستخدم Function ‏`std::mem::drop`.

تختلف Function ‏`std::mem::drop` عن Method ‏`drop` في `Drop` Trait. نستدعيها عن طريق تمرير القيمة التي نريد فرض إسقاطها كوسيط (Argument). توجد Function في Prelude، لذا يمكننا تعديل `main` في القائمة 15-15 لاستدعاء Function ‏`drop` كما هو موضح في القائمة 15-16.

<Listing number="15-16" file-name="src/main.rs" caption="استدعاء `std::mem::drop` لإسقاط قيمة صراحة قبل خروجها من النطاق">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-16/src/main.rs:here}}
```

</Listing>

سيؤدي تشغيل هذا Code إلى طباعة ما يلي:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-16/output.txt}}
```

تتم طباعة النص ``Dropping CustomSmartPointer with data `some data`!`` بين نص `CustomSmartPointer created` و `CustomSmartPointer dropped before the end of main` مما يوضح أنه يتم استدعاء Code الخاص بـ Method ‏`drop` لإسقاط `c` عند تلك النقطة.

يمكنك استخدام Code المحدد في تنفيذ `Drop` Trait بعدة طرق لجعل التنظيف مريحاً وآمناً: على سبيل المثال، يمكنك استخدامه لإنشاء مخصص ذاكرة (Memory Allocator) خاص بك! مع `Drop` Trait ونظام الملكية (Ownership System) في Rust، لا يتعين عليك تذكر القيام بالتنظيف، لأن Rust تقوم بذلك تلقائياً.

كما لا داعي للقلق بشأن المشكلات الناتجة عن تنظيف القيم التي لا تزال قيد الاستخدام عن طريق الخطأ: فنظام الملكية الذي يضمن أن References صالحة دائماً يضمن أيضاً استدعاء `drop` مرة واحدة فقط عندما لا تعود القيمة قيد الاستخدام.

الآن بعد أن فحصنا `Box<T>` وبعض خصائص Smart Pointers، دعنا نلقي نظرة على عدد قليل من Smart Pointers الأخرى المعرفة في Standard Library.

[ch13]: ch13-00-functional-features.html
[ch7-idiomatic-use]: ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#creating-idiomatic-use-paths
