## فصل الوحدات البرمجية في ملفات مختلفة

حتى الآن، كانت جميع الأمثلة في هذا الفصل تعرف وحدات برمجية (modules) متعددة في ملف واحد. عندما تصبح modules كبيرة، قد ترغب في نقل تعريفاتها إلى ملف منفصل لتسهيل تصفح الكود.

على سبيل المثال، لنبدأ من الكود الموجود في القائمة 7-17 الذي كان يحتوي على عدة modules للمطعم. سنقوم باستخراج modules إلى ملفات بدلاً من وجود جميع modules معرفة في ملف جذر الكريت (crate root). في هذه الحالة، ملف crate root هو _src/lib.rs_، ولكن هذا الإجراء يعمل أيضاً مع الكريتات الثنائية (binary crates) التي يكون ملف crate root الخاص بها هو _src/main.rs_.

أولاً، سنقوم باستخراج وحدة `front_of_house` إلى ملفها الخاص. قم بإزالة الكود الموجود داخل الأقواس المتعرجة لوحدة `front_of_house` مع ترك التصريح `mod front_of_house;` فقط، بحيث يحتوي _src/lib.rs_ على الكود الموضح في القائمة 7-21. لاحظ أن هذا لن يتم تصريفه (compile) حتى ننشئ ملف _src/front_of_house.rs_ في القائمة 7-22.

<Listing number="7-21" file-name="src/lib.rs" caption="التصريح عن وحدة `front_of_house` التي سيكون جسمها في *src/front_of_house.rs*">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-21-and-22/src/lib.rs}}
```

</Listing>

بعد ذلك، ضع الكود الذي كان داخل الأقواس المتعرجة في ملف جديد باسم _src/front_of_house.rs_، كما هو موضح في القائمة 7-22. يعرف المترجم (compiler) أنه يجب البحث في هذا الملف لأنه صادف تصريح الوحدة في crate root باسم `front_of_house`.

<Listing number="7-22" file-name="src/front_of_house.rs" caption="التعريفات داخل وحدة `front_of_house` في *src/front_of_house.rs*">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-21-and-22/src/front_of_house.rs}}
```

</Listing>

لاحظ أنك تحتاج فقط إلى تحميل ملف باستخدام تصريح `mod` _مرة واحدة_ في شجرة الوحدات البرمجية (module tree). بمجرد أن يعرف compiler أن الملف جزء من المشروع (ويعرف مكان وجود الكود في module tree بسبب المكان الذي وضعت فيه عبارة `mod`)، يجب أن تشير الملفات الأخرى في مشروعك إلى كود الملف المحمل باستخدام مسار (path) إلى مكان التصريح عنه، كما هو مغطى في قسم ["مسارات الإشارة إلى عنصر في شجرة الوحدات البرمجية"][paths]. بعبارة أخرى، `mod` _ليست_ عملية "تضمين" (include) كما قد تكون رأيتها في لغات برمجة أخرى.

بعد ذلك، سنقوم باستخراج وحدة `hosting` إلى ملفها الخاص. العملية مختلفة قليلاً لأن `hosting` هي وحدة فرعية (child module) لـ `front_of_house` وليست للوحدة الجذرية. سنضع ملف `hosting` في مجلد جديد سيتم تسميته باسم أسلافه في module tree، وفي هذه الحالة هو _src/front_of_house_.

لبدء نقل `hosting` نقوم بتغيير _src/front_of_house.rs_ ليحتوي فقط على تصريح وحدة `hosting`:

<Listing file-name="src/front_of_house.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-02-extracting-hosting/src/front_of_house.rs}}
```

</Listing>

ثم ننشئ مجلد _src/front_of_house_ وملف _hosting.rs_ ليحتويا على التعريفات الموجودة في وحدة `hosting`:

<Listing file-name="src/front_of_house/hosting.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-02-extracting-hosting/src/front_of_house/hosting.rs}}
```

</Listing>

إذا وضعنا _hosting.rs_ في مجلد _src_ بدلاً من ذلك، فسيوقع compiler أن يكون كود _hosting.rs_ في وحدة `hosting` مصرح عنها في crate root وليس مصرحاً عنها كابن لوحدة `front_of_house`. تعني قواعد compiler الخاصة بالملفات التي يجب فحصها بحثاً عن كود modules أن المجلدات والملفات تتطابق بشكل أوثق مع module tree.

> ### مسارات ملفات بديلة (Alternate File Paths)
>
> حتى الآن قمنا بتغطية مسارات الملفات الأكثر شيوعاً (idiomatic) التي يستخدمها مترجم Rust، ولكن Rust يدعم أيضاً أسلوباً قديماً لمسارات الملفات. بالنسبة لوحدة تسمى `front_of_house` مصرح عنها في crate root، سيبحث compiler عن كود الوحدة في:
>
> - _src/front_of_house.rs_ (ما قمنا بتغطيته)
> - _src/front_of_house/mod.rs_ (أسلوب قديم، مسار لا يزال مدعوماً)
>
> بالنسبة لوحدة تسمى `hosting` وهي وحدة فرعية (submodule) لـ `front_of_house` سيبحث compiler عن كود الوحدة في:
>
> - _src/front_of_house/hosting.rs_ (ما قمنا بتغطيته)
> - _src/front_of_house/hosting/mod.rs_ (أسلوب قديم، مسار لا يزال مدعوماً)
>
> إذا استخدمت كلا الأسلوبين لنفس الوحدة، فستحصل على خطأ من المترجم (compiler error). يسمح باستخدام مزيج من كلا الأسلوبين لوحدات مختلفة في نفس المشروع ولكن قد يكون ذلك مربكاً للأشخاص الذين يتصفحون مشروعك.
>
> العيب الرئيسي للأسلوب الذي يستخدم ملفات تسمى _mod.rs_ هو أن مشروعك قد ينتهي به الأمر بالعديد من الملفات المسماة _mod.rs_، مما قد يصبح مربكاً عندما تكون مفتوحة في المحرر الخاص بك في نفس الوقت.

لقد نقلنا كود كل وحدة إلى ملف منفصل، وظلت module tree كما هي. ستعمل استدعاءات الدوال في `eat_at_restaurant` دون أي تعديل، على الرغم من أن التعريفات تعيش في ملفات مختلفة. تتيح لك هذه التقنية نقل modules إلى ملفات جديدة مع زيادة حجمها.

لاحظ أن عبارة `pub use crate::front_of_house::hosting` في _src/lib.rs_ لم تتغير أيضاً، كما أن `use` ليس لها أي تأثير على الملفات التي يتم تصريفها كجزء من crate. تقوم الكلمة المفتاحية `mod` بالتصريح عن modules، ويبحث Rust في ملف له نفس اسم الوحدة عن الكود الذي يوضع في تلك الوحدة.

## ملخص

تسمح لك Rust بتقسيم طرد (package) إلى عدة كريتات (crates) والكريت إلى وحدات برمجية (modules) بحيث يمكنك الرجوع إلى العناصر المعرفة في وحدة من وحدة أخرى. يمكنك القيام بذلك عن طريق تحديد مسارات (paths) مطلقة أو نسبية. يمكن جلب هذه paths إلى النطاق (scope) باستخدام عبارة `use` بحيث يمكنك استخدام مسار أقصر لاستخدامات متعددة للعنصر في ذلك scope. كود الوحدة يكون خاصاً (private) بشكل افتراضي، ولكن يمكنك جعل التعريفات عامة (public) بإضافة الكلمة المفتاحية `pub`.

في الفصل القادم، سنلقي نظرة على بعض هياكل بيانات المجموعات (collection data structures) في المكتبة القياسية (standard library) التي يمكنك استخدامها في كودك المنظم بدقة.

[paths]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
