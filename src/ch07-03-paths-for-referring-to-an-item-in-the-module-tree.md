## المسارات (Paths) للإشارة إلى عنصر في شجرة الوحدات (Module Tree)

لإظهار لغة Rust أين تجد عنصرًا في شجرة الوحدات (module tree)، نستخدم مسارًا (path) بنفس الطريقة التي نستخدم بها مسارًا عند التنقل في نظام الملفات (filesystem). لاستدعاء دالة (function)، نحتاج إلى معرفة مسارها.

يمكن أن يتخذ المسار شكلين:

- *المسار المطلق* (Absolute path) هو المسار الكامل الذي يبدأ من جذر الصندوق (crate root)؛ بالنسبة للكود من صندوق خارجي (external crate)، يبدأ المسار المطلق باسم الـ crate، وبالنسبة للكود من الـ crate الحالي، يبدأ بالحرف `crate`.
- *المسار النسبي* (Relative path) يبدأ من الوحدة (module) الحالية ويستخدم `self` أو `super` أو معرفًا (identifier) في الـ module الحالي.

يتبع كل من الـ absolute paths والـ relative paths بمعرف واحد أو أكثر مفصول بعلامتي نقطتين مزدوجتين (`::`).

بالعودة إلى القائمة 7-1، لنفترض أننا نريد استدعاء الدالة `add_to_waitlist`. هذا هو نفس السؤال: ما هو مسار الدالة `add_to_waitlist`؟ تحتوي القائمة 7-3 على القائمة 7-1 مع إزالة بعض الـ modules والدوال.

سنعرض طريقتين لاستدعاء الدالة `add_to_waitlist` من دالة جديدة، `eat_at_restaurant`، محددة في الـ crate root. هذه المسارات صحيحة، ولكن هناك مشكلة أخرى متبقية ستمنع هذا المثال من الـ compile كما هو. سنشرح السبب بعد قليل.

تعد الدالة `eat_at_restaurant` جزءًا من واجهة برمجة التطبيقات العامة (public API) لـ library crate الخاص بنا، لذلك نضع علامة عليها بالكلمة المفتاحية `pub`. في قسم ["كشف المسارات باستخدام الكلمة المفتاحية `pub`"][pub]، سنتعمق في تفاصيل أكثر حول `pub`.

<Listing number="7-3" file-name="src/lib.rs" caption="استدعاء الدالة `add_to_waitlist` باستخدام المسارات المطلقة والنسبية">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-03/src/lib.rs}}
```

</Listing>

في المرة الأولى التي نستدعي فيها الدالة `add_to_waitlist` في `eat_at_restaurant`، نستخدم absolute path. تم تعريف الدالة `add_to_waitlist` في نفس الـ crate مثل `eat_at_restaurant`، مما يعني أنه يمكننا استخدام الكلمة المفتاحية `crate` لبدء absolute path. ثم نقوم بتضمين كل من الـ modules المتتالية حتى نصل إلى `add_to_waitlist`. يمكنك تخيل filesystem بنفس الهيكل: سنحدد المسار `/front_of_house/hosting/add_to_waitlist` لتشغيل برنامج `add_to_waitlist`؛ استخدام اسم الـ crate للبدء من الـ crate root يشبه استخدام `/` للبدء من جذر نظام الملفات في الـ shell الخاص بك.

في المرة الثانية التي نستدعي فيها `add_to_waitlist` في `eat_at_restaurant`، نستخدم relative path. يبدأ المسار بـ `front_of_house`، وهو اسم الـ module المحدد في نفس مستوى الـ module tree مثل `eat_at_restaurant`. هنا، سيكون مكافئ نظام الملفات هو استخدام المسار `front_of_house/hosting/add_to_waitlist`. البدء باسم module يعني أن المسار نسبي.

إن اختيار ما إذا كنت ستستخدم relative path أو absolute path هو قرار ستتخذه بناءً على مشروعك، ويعتمد على ما إذا كنت أكثر عرضة لنقل كود تعريف العنصر بشكل منفصل عن الكود الذي يستخدم العنصر أو معه. على سبيل المثال، إذا نقلنا الـ module `front_of_house` والدالة `eat_at_restaurant` إلى module يسمى `customer_experience`، فسنحتاج إلى تحديث الـ absolute path إلى `add_to_waitlist`، لكن الـ relative path سيظل صالحًا. ومع ذلك، إذا نقلنا الدالة `eat_at_restaurant` بشكل منفصل إلى module يسمى `dining`، فسيظل الـ absolute path إلى استدعاء `add_to_waitlist` كما هو، ولكن سيحتاج الـ relative path إلى التحديث. تفضيلنا بشكل عام هو تحديد absolute paths لأنه من المرجح أننا سنرغب في نقل تعريفات الكود واستدعاءات العناصر بشكل مستقل عن بعضها البعض.

دعنا نحاول compile القائمة 7-3 ونكتشف سبب عدم الـ compile بعد! تظهر الأخطاء التي نحصل عليها في القائمة 7-4.

<Listing number="7-4" caption="أخطاء المترجم من بناء الكود في القائمة 7-3">

```console
{{#include ../listings/ch07-managing-growing-projects/listing-07-03/output.txt}}
```

</Listing>

تقول رسائل الخطأ أن الـ module `hosting` خاص (private). بعبارة أخرى، لدينا المسارات الصحيحة لـ module `hosting` والدالة `add_to_waitlist`، لكن Rust لن تسمح لنا باستخدامها لأنه ليس لديها وصول إلى الأقسام الـ private. في Rust، تكون جميع العناصر (الدوال، الـ methods، الـ structs، الـ enums، الـ modules، والثوابت) خاصة بـ parent modules افتراضيًا. إذا كنت تريد جعل عنصر مثل function أو struct خاصًا، فستضعه في module.

لا يمكن للعناصر الموجودة في parent module استخدام العناصر الـ private داخل child modules، ولكن يمكن للعناصر الموجودة في child modules استخدام العناصر الموجودة في ancestor modules الخاصة بها. هذا لأن الـ child modules تغلف وتخفي تفاصيل التطبيق الخاصة بها، ولكن يمكن لـ child modules رؤية السياق الذي تم تعريفها فيه. لمواصلة استعارتنا، فكر في قواعد الخصوصية على أنها مثل المكتب الخلفي للمطعم: ما يحدث هناك خاص لعملاء المطعم، ولكن يمكن لمديري المكاتب رؤية والقيام بكل شيء في المطعم الذي يديرونه.

اختارت Rust أن يعمل نظام الـ module بهذه الطريقة بحيث يكون إخفاء تفاصيل التطبيق الداخلية هو الافتراضي. بهذه الطريقة، تعرف أي أجزاء من الكود الداخلي يمكنك تغييرها دون كسر الكود الخارجي. ومع ذلك، تمنحك Rust خيار كشف الأجزاء الداخلية من كود الـ child modules لـ ancestor modules الخارجية باستخدام الكلمة المفتاحية `pub` لجعل العنصر عامًا (public).

### كشف المسارات باستخدام الكلمة المفتاحية `pub`

دعنا نعود إلى الخطأ في القائمة 7-4 الذي أخبرنا أن الـ module `hosting` خاص. نريد أن يكون للدالة `eat_at_restaurant` في الـ parent module وصول إلى الدالة `add_to_waitlist` في الـ child module، لذلك نضع علامة على الـ module `hosting` بالكلمة المفتاحية `pub`، كما هو موضح في القائمة 7-5.

<Listing number="7-5" file-name="src/lib.rs" caption="الإعلان عن الـ module `hosting` كـ `pub` لاستخدامه من `eat_at_restaurant`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-05/src/lib.rs:here}}
```

</Listing>

لسوء الحظ، لا يزال الكود في القائمة 7-5 يؤدي إلى أخطاء في الـ compiler، كما هو موضح في القائمة 7-6.

<Listing number="7-6" caption="أخطاء المترجم من بناء الكود في القائمة 7-5">

```console
{{#include ../listings/ch07-managing-growing-projects/listing-07-05/output.txt}}
```

</Listing>

ماذا حدث؟ إضافة الكلمة المفتاحية `pub` أمام `mod hosting` تجعل الـ module عامًا. مع هذا التغيير، إذا تمكنا من الوصول إلى `front_of_house`، فيمكننا الوصول إلى `hosting`. لكن *محتويات* `hosting` لا تزال خاصة؛ جعل الـ module عامًا لا يجعل محتوياته عامة. تسمح الكلمة المفتاحية `pub` على module فقط للكود الموجود في ancestor modules بالإشارة إليه، وليس الوصول إلى الكود الداخلي الخاص به. نظرًا لأن الـ modules هي حاويات، فلا يمكننا فعل الكثير بمجرد جعل الـ module عامًا؛ نحتاج إلى المضي قدمًا واختيار جعل عنصر واحد أو أكثر داخل الـ module عامًا أيضًا.

تقول الأخطاء في القائمة 7-6 أن الدالة `add_to_waitlist` خاصة. تنطبق قواعد الخصوصية على الـ structs، الـ enums، الـ functions، والـ methods بالإضافة إلى الـ modules.

دعنا نجعل الدالة `add_to_waitlist` عامة أيضًا عن طريق إضافة الكلمة المفتاحية `pub` قبل تعريفها، كما في القائمة 7-7.

<Listing number="7-7" file-name="src/lib.rs" caption="إضافة الكلمة المفتاحية `pub` إلى `mod hosting` و `fn add_to_waitlist` يسمح لنا باستدعاء الدالة من `eat_at_restaurant`.">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-07/src/lib.rs:here}}
```

</Listing>

الآن سيتم الـ compile للكود! لمعرفة سبب سماح إضافة الكلمة المفتاحية `pub` لنا باستخدام هذه المسارات في `eat_at_restaurant` فيما يتعلق بقواعد الخصوصية، دعنا ننظر إلى الـ absolute paths والـ relative paths.

في الـ absolute path، نبدأ بـ `crate`، وهو جذر الـ module tree لـ crate الخاص بنا. تم تعريف الـ module `front_of_house` في الـ crate root. على الرغم من أن `front_of_house` ليس عامًا، نظرًا لأن الدالة `eat_at_restaurant` محددة في نفس الـ module مثل `front_of_house` (أي أن `eat_at_restaurant` و `front_of_house` هما شقيقان)، يمكننا الإشارة إلى `front_of_house` من `eat_at_restaurant`. التالي هو الـ module `hosting` الذي تم وضع علامة عليه بـ `pub`. يمكننا الوصول إلى الـ parent module لـ `hosting`، لذلك يمكننا الوصول إلى `hosting`. أخيرًا، تم وضع علامة على الدالة `add_to_waitlist` بـ `pub`، ويمكننا الوصول إلى الـ parent module الخاص بها، لذا فإن استدعاء الدالة هذا يعمل!

في الـ relative path، يكون المنطق هو نفسه الـ absolute path باستثناء الخطوة الأولى: بدلاً من البدء من الـ crate root، يبدأ المسار من `front_of_house`. تم تعريف الـ module `front_of_house` داخل نفس الـ module مثل `eat_at_restaurant`، لذا فإن الـ relative path الذي يبدأ من الـ module الذي تم تعريف `eat_at_restaurant` فيه يعمل. بعد ذلك، نظرًا لأن `hosting` و `add_to_waitlist` تم وضع علامة عليهما بـ `pub`، فإن بقية المسار يعمل، واستدعاء الدالة هذا صالح!

إذا كنت تخطط لمشاركة library crate الخاص بك بحيث يمكن للمشاريع الأخرى استخدام الكود الخاص بك، فإن الـ public API الخاص بك هو عقدك مع مستخدمي الـ crate الخاص بك الذي يحدد كيفية تفاعلهم مع الكود الخاص بك. هناك العديد من الاعتبارات حول إدارة التغييرات على الـ public API الخاص بك لتسهيل اعتماد الأشخاص على الـ crate الخاص بك. هذه الاعتبارات تتجاوز نطاق هذا الكتاب؛ إذا كنت مهتمًا بهذا الموضوع، فراجع [إرشادات Rust API][api-guidelines].

> #### أفضل الممارسات للحزم التي تحتوي على ثنائي ومكتبة (Binary and a Library)
>
> ذكرنا أن الحزمة يمكن أن تحتوي على كل من جذر الـ binary crate `src/main.rs` بالإضافة إلى جذر الـ library crate `src/lib.rs`، وسيكون لكلا الـ crates اسم الحزمة افتراضيًا. عادةً، تحتوي الحزم التي تحتوي على هذا النمط من احتواء كل من library crate و binary crate على كود كافٍ فقط في الـ binary crate لبدء ملف تنفيذي يستدعي الكود المحدد في الـ library crate. يتيح ذلك للمشاريع الأخرى الاستفادة من معظم الوظائف التي توفرها الحزمة لأنه يمكن مشاركة كود الـ library crate.
>
> يجب تعريف الـ module tree في `src/lib.rs`. بعد ذلك، يمكن استخدام أي عناصر عامة في الـ binary crate عن طريق بدء المسارات باسم الحزمة. يصبح الـ binary crate مستخدمًا لـ library crate تمامًا مثلما يستخدم الـ external crate بالكامل الـ library crate: يمكنه فقط استخدام الـ public API. يساعدك هذا في تصميم API جيد؛ لست أنت المؤلف فحسب، بل أنت أيضًا عميل!
>
> في [الفصل 12][ch12]، سنوضح هذه الممارسة التنظيمية باستخدام برنامج سطر أوامر سيحتوي على كل من binary crate و library crate.

### بدء الـ Relative Paths بـ `super`

يمكننا إنشاء relative paths تبدأ في الـ parent module، بدلاً من الـ module الحالي أو الـ crate root، باستخدام `super` في بداية المسار. هذا يشبه بدء مسار filesystem ببناء جملة `..` الذي يعني الانتقال إلى الدليل الأصل (parent directory). يسمح لنا استخدام `super` بالإشارة إلى عنصر نعرف أنه موجود في الـ parent module، مما قد يجعل إعادة ترتيب الـ module tree أسهل عندما يكون الـ module مرتبطًا ارتباطًا وثيقًا بالـ parent ولكن قد يتم نقل الـ parent إلى مكان آخر في الـ module tree في يوم من الأيام.

ضع في اعتبارك الكود في القائمة 7-8 الذي يصمم الموقف الذي يقوم فيه طاهٍ بإصلاح طلب غير صحيح ويحضره شخصيًا إلى العميل. تستدعي الدالة `fix_incorrect_order` المحددة في الـ module `back_of_house` الدالة `deliver_order` المحددة في الـ parent module عن طريق تحديد المسار إلى `deliver_order`، بدءًا من `super`.

<Listing number="7-8" file-name="src/lib.rs" caption="استدعاء دالة باستخدام relative path يبدأ بـ `super`">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-08/src/lib.rs}}
```

</Listing>

توجد الدالة `fix_incorrect_order` في الـ module `back_of_house`، لذلك يمكننا استخدام `super` للانتقال إلى الـ parent module لـ `back_of_house`، وهو في هذه الحالة `crate`، الجذر. من هناك، نبحث عن `deliver_order` ونجده. نجاح! نعتقد أن الـ module `back_of_house` والدالة `deliver_order` من المرجح أن يظلا في نفس العلاقة مع بعضهما البعض ويتم نقلهما معًا إذا قررنا إعادة تنظيم الـ module tree لـ crate. لذلك، استخدمنا `super` حتى يكون لدينا عدد أقل من الأماكن لتحديث الكود فيها في المستقبل إذا تم نقل هذا الكود إلى module مختلف.

### جعل الـ Structs والـ Enums عامة

يمكننا أيضًا استخدام `pub` لتعيين الـ structs والـ enums كـ public، ولكن هناك بعض الاختلافات. إذا وضعنا `pub` قبل تعريف struct، فإننا نجعل الـ struct عامًا، لكن الـ fields الخاصة بالـ struct ستظل خاصة. يمكننا جعل كل field عامًا أو لا على أساس كل حالة على حدة. في القائمة 7-9، قمنا بتعريف struct عام `back_of_house::Breakfast` مع field عام `toast` ولكن field خاص `seasonal_fruit`. يصمم هذا الحالة في مطعم حيث يمكن للعميل اختيار نوع الخبز الذي يأتي مع الوجبة، لكن الطاهي يقرر الفاكهة التي تصاحب الوجبة بناءً على ما هو موسمي ومتوفر. تتغير الفاكهة المتاحة بسرعة، لذلك لا يمكن للعملاء اختيار الفاكهة أو حتى رؤية الفاكهة التي سيحصلون عليها.

<Listing number="7-9" file-name="src/lib.rs" caption="struct به بعض الـ fields العامة وبعض الـ fields الخاصة">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-09/src/lib.rs}}
```

</Listing>

نظرًا لأن الـ field `toast` في struct `back_of_house::Breakfast` عام، في `eat_at_restaurant` يمكننا الكتابة والقراءة إلى الـ field `toast` باستخدام تدوين النقطة (dot notation). لاحظ أنه لا يمكننا استخدام الـ field `seasonal_fruit` في `eat_at_restaurant`، لأن `seasonal_fruit` خاص. حاول إلغاء التعليق على السطر الذي يعدل قيمة الـ field `seasonal_fruit` لترى الخطأ الذي تحصل عليه!

لاحظ أيضًا أنه نظرًا لأن `back_of_house::Breakfast` يحتوي على field خاص، يحتاج الـ struct إلى توفير دالة مرتبطة عامة تنشئ مثيلًا لـ `Breakfast` (لقد أطلقنا عليها اسم `summer` هنا). إذا لم يكن لدى `Breakfast` مثل هذه الدالة، فلن نتمكن من إنشاء مثيل لـ `Breakfast` في `eat_at_restaurant`، لأننا لا يمكننا تعيين قيمة الـ field الخاص `seasonal_fruit` في `eat_at_restaurant`.

في المقابل، إذا جعلنا enum عامًا، فإن جميع متغيراته (variants) تكون عامة. نحتاج فقط إلى `pub` قبل الكلمة المفتاحية `enum`، كما هو موضح في القائمة 7-10.

<Listing number="7-10" file-name="src/lib.rs" caption="تعيين enum كـ public يجعل جميع متغيراته عامة.">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-10/src/lib.rs}}
```

</Listing>

نظرًا لأننا جعلنا الـ enum `Appetizer` عامًا، يمكننا استخدام الـ variants `Soup` و `Salad` في `eat_at_restaurant`.

الـ Enums ليست مفيدة جدًا ما لم تكن متغيراتها عامة؛ سيكون من المزعج الاضطرار إلى إضافة تعليق توضيحي لجميع enum variants بـ `pub` في كل حالة، لذا فإن الإعداد الافتراضي لـ enum variants هو أن تكون عامة. غالبًا ما تكون الـ Structs مفيدة دون أن تكون الـ fields الخاصة بها عامة، لذا تتبع الـ struct fields القاعدة العامة المتمثلة في أن كل شيء خاص افتراضيًا ما لم يتم وضع علامة عليه بـ `pub`.

هناك موقف آخر يتعلق بـ `pub` لم نقم بتغطيته، وهو آخر ميزة لنظام الـ module: الكلمة المفتاحية `use`. سنغطي `use` بمفردها أولاً، ثم سنوضح كيفية دمج `pub` و `use`.

[pub]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#exposing-paths-with-the-pub-keyword
[api-guidelines]: https://rust-lang.github.io/api-guidelines/
[ch12]: ch12-00-an-io-project.html
