## تخزين المفاتيح مع القيم المرتبطة في الخرائط الهاشية (Hash Maps)

الخريطة الهاشية (Hash Map) هي آخر مجموعة شائعة لدينا. يخزن النوع `HashMap<K, V>` تعيينًا للمفاتيح (keys) من النوع `K` إلى القيم (values) من النوع `V` باستخدام دالة تجزئة (hashing function)، والتي تحدد كيفية وضع هذه keys و values في الذاكرة. تدعم العديد من لغات البرمجة هذا النوع من هياكل البيانات، ولكنها غالبًا ما تستخدم اسمًا مختلفًا، مثل hash، map، object، hash table، dictionary، أو associative array، على سبيل المثال لا الحصر.

تعد Hash Maps مفيدة عندما تريد البحث عن البيانات ليس باستخدام فهرس، كما يمكنك أن تفعل مع المتجهات (vectors)، ولكن باستخدام key يمكن أن يكون من أي نوع. على سبيل المثال، في لعبة ما، يمكنك تتبع نتيجة كل فريق في Hash Map يكون فيها كل key هو اسم الفريق و values هي نتيجة كل فريق. بالنظر إلى اسم الفريق، يمكنك استرداد نتيجته.

سنتناول واجهة برمجة التطبيقات (API) الأساسية لـ Hash Maps في هذا القسم، ولكن هناك المزيد من الميزات مخبأة في الدوال المعرفة على `HashMap<K, V>` بواسطة المكتبة القياسية (standard library). كما هو الحال دائمًا، تحقق من وثائق standard library لمزيد من المعلومات.

### إنشاء Hash Map جديدة

إحدى طرق إنشاء Hash Map فارغة هي استخدام `new` وإضافة العناصر باستخدام `insert`. في القائمة 8-20، نتتبع نتائج فريقين هما _الأزرق_ و_الأصفر_. يبدأ الفريق الأزرق بـ 10 نقاط، والفريق الأصفر بـ 50.

<Listing number="8-20" caption="Creating a new hash map and inserting some keys and values">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-20/src/main.rs:here}}
```

</Listing>

لاحظ أننا نحتاج أولاً إلى `use` لـ `HashMap` من جزء المجموعات في standard library. من بين مجموعاتنا الشائعة الثلاث، هذه هي الأقل استخدامًا، لذا فهي غير مدرجة في الميزات التي يتم جلبها تلقائيًا إلى النطاق (scope) في المقدمة (prelude). تحظى Hash Maps أيضًا بدعم أقل من standard library؛ لا يوجد ماكرو (macro) مدمج لإنشائها، على سبيل المثال.

تمامًا مثل vectors، تخزن Hash Maps بياناتها على الكومة (heap). تحتوي `HashMap` هذه على keys من النوع سلسلة نصية (`String`) و values من النوع عدد صحيح 32 بت (`i32`). مثل vectors، فإن Hash Maps متجانسة (homogeneous): يجب أن تكون جميع keys من نفس النوع، ويجب أن تكون جميع values من نفس النوع.

### الوصول إلى القيم في Hash Map

يمكننا الحصول على value من Hash Map عن طريق توفير key الخاص بها لدالة `get`، كما هو موضح في القائمة 8-21.

<Listing number="8-21" caption="Accessing the score for the Blue team stored in the hash map">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-21/src/main.rs:here}}
```

</Listing>

هنا، سيكون لـ `score` القيمة المرتبطة بالفريق الأزرق، وستكون النتيجة `10`. تُرجع دالة `get` خيارًا (`Option<&V>`)؛ إذا لم تكن هناك value لهذا key في Hash Map، فستُرجع `get` لا شيء (`None`). يتعامل هذا البرنامج مع `Option` عن طريق استدعاء `copied` للحصول على `Option<i32>` بدلاً من `Option<&i32>`، ثم `unwrap_or` لتعيين `score` إلى صفر إذا لم يكن لدى `scores` إدخال (entry) لـ key.

يمكننا التكرار (iterate) على كل زوج key-value في Hash Map بطريقة مماثلة لما نفعله مع vectors، باستخدام حلقة `for`:

```rust
{{#rustdoc_include ../listings/ch08-common-collections/no-listing-03-iterate-over-hashmap/src/main.rs:here}}
```

سيقوم هذا الكود بطباعة كل زوج بترتيب عشوائي:

```text
Yellow: 50
Blue: 10
```

<!-- Old headings. Do not remove or links may break. -->

<a id="hash-maps-and-ownership"></a>

### إدارة الملكية (Ownership) في Hash Maps

بالنسبة للأنواع التي تطبق سمة النسخ (`Copy trait`)، مثل `i32`، يتم نسخ values إلى Hash Map. بالنسبة للقيم المملوكة (owned values) مثل `String`، سيتم نقل values وستكون Hash Map هي مالك (owner) تلك values، كما هو موضح في القائمة 8-22.

<Listing number="8-22" caption="Showing that keys and values are owned by the hash map once they’re inserted">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-22/src/main.rs:here}}
```

</Listing>

لا يمكننا استخدام المتغيرين `field_name` و `field_value` بعد نقلهما إلى Hash Map باستدعاء `insert`.

إذا قمنا بإدراج مراجع (references) إلى values في Hash Map، فلن يتم نقل values إلى Hash Map. يجب أن تكون values التي تشير إليها references صالحة على الأقل طالما أن Hash Map صالحة. سنتحدث أكثر عن هذه المشكلات في [“Validating References with Lifetimes”][validating-references-with-lifetimes]<!-- ignore --> في الفصل 10.

### تحديث Hash Map

على الرغم من أن عدد أزواج key و value قابل للنمو، إلا أن كل key فريد يمكن أن يكون له value واحدة فقط مرتبطة به في كل مرة (ولكن ليس العكس: على سبيل المثال، يمكن أن يكون لكل من الفريق الأزرق والفريق الأصفر القيمة `10` مخزنة في Hash Map `scores`).

عندما تريد تغيير البيانات في Hash Map، عليك أن تقرر كيفية التعامل مع الحالة التي يكون فيها لـ key قيمة معينة بالفعل. يمكنك استبدال value القديمة بـ value الجديدة، متجاهلاً value القديمة تمامًا. يمكنك الاحتفاظ بـ value القديمة وتجاهل value الجديدة، وإضافة value الجديدة فقط إذا لم يكن لـ key قيمة بالفعل. أو يمكنك دمج value القديمة و value الجديدة. دعونا نرى كيف نفعل كل واحدة من هذه!

#### الكتابة فوق قيمة (Overwriting a Value)

إذا قمنا بإدراج key و value في Hash Map ثم قمنا بإدراج نفس key بـ value مختلفة، فسيتم استبدال value المرتبطة بهذا key. على الرغم من أن الكود في القائمة 8-23 يستدعي `insert` مرتين، إلا أن Hash Map ستحتوي على زوج key-value واحد فقط لأننا نقوم بإدراج value لـ key الفريق الأزرق في كلتا المرتين.

<Listing number="8-23" caption="Replacing a value stored with a particular key">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-23/src/main.rs:here}}
```

</Listing>

سيقوم هذا الكود بطباعة `{"Blue": 25}`. تم الكتابة فوق (overwritten) القيمة الأصلية `10`.

<!-- Old headings. Do not remove or links may break. -->

<a id="only-inserting-a-value-if-the-key-has-no-value"></a>

#### إضافة Key و Value فقط إذا لم يكن Key موجودًا

من الشائع التحقق مما إذا كان key معين موجودًا بالفعل في Hash Map بقيمة، ثم اتخاذ الإجراءات التالية: إذا كان key موجودًا في Hash Map، فيجب أن تظل value الحالية كما هي؛ إذا لم يكن key موجودًا، فقم بإدراجه و value له.

تحتوي Hash Maps على واجهة API خاصة لذلك تسمى `entry` والتي تأخذ key الذي تريد التحقق منه كمعامل. القيمة المرجعة لدالة `entry` هي تعداد (enum) يسمى `Entry` يمثل value قد تكون موجودة أو لا تكون موجودة. لنفترض أننا نريد التحقق مما إذا كان key للفريق الأصفر له value مرتبطة به. إذا لم يكن كذلك، فنريد إدراج value `50`، ونفس الشيء للفريق الأزرق. باستخدام واجهة API لـ `entry`، يبدو الكود كما في القائمة 8-24.

<Listing number="8-24" caption="Using the `entry` method to only insert if the key does not already have a value">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-24/src/main.rs:here}}
```

</Listing>

تم تعريف دالة `or_insert` على `Entry` لإرجاع مرجع قابل للتغيير (`mutable reference`) إلى value لـ key الخاص بـ `Entry` المقابل إذا كان هذا key موجودًا، وإذا لم يكن كذلك، فإنه يدرج المعامل كـ value الجديدة لهذا key ويعيد mutable reference إلى value الجديدة. هذه التقنية أنظف بكثير من كتابة المنطق بأنفسنا، وبالإضافة إلى ذلك، تتوافق بشكل أفضل مع مدقق الاستعارة (`borrow checker`).

سيؤدي تشغيل الكود في القائمة 8-24 إلى طباعة `{"Yellow": 50, "Blue": 10}`. سيقوم الاستدعاء الأول لـ `entry` بإدراج key للفريق الأصفر بـ value `50` لأن الفريق الأصفر ليس لديه value بالفعل. لن يغير الاستدعاء الثاني لـ `entry` Hash Map، لأن الفريق الأزرق لديه value `10` بالفعل.

#### تحديث قيمة بناءً على القيمة القديمة

حالة استخدام شائعة أخرى لـ Hash Maps هي البحث عن value لـ key ثم تحديثها بناءً على value القديمة. على سبيل المثال، تعرض القائمة 8-25 كودًا يحسب عدد مرات ظهور كل كلمة في نص ما. نستخدم Hash Map مع الكلمات كـ keys ونزيد value لتتبع عدد المرات التي رأينا فيها تلك الكلمة. إذا كانت هذه هي المرة الأولى التي نرى فيها كلمة، فسنقوم أولاً بإدراج value `0`.

<Listing number="8-25" caption="Counting occurrences of words using a hash map that stores words and counts">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-25/src/main.rs:here}}
```

</Listing>

سيقوم هذا الكود بطباعة `{"world": 2, "hello": 1, "wonderful": 1}`. قد ترى أزواج key-value نفسها مطبوعة بترتيب مختلف: تذكر من [“Accessing Values in a Hash Map”][access]<!-- ignore --> أن التكرار على Hash Map يحدث بترتيب عشوائي.

تُرجع دالة `split_whitespace` مكررًا (iterator) على الشرائح الفرعية (subslices)، مفصولة بمسافات بيضاء، لـ value في `text`. تُرجع دالة `or_insert` مرجعًا قابلًا للتغيير (`&mut V`) إلى value لـ key المحدد. هنا، نقوم بتخزين هذا mutable reference في المتغير `count`، لذلك من أجل التعيين لتلك value، يجب علينا أولاً فك الإشارة (dereference) لـ `count` باستخدام النجمة (`*`). يخرج mutable reference من النطاق في نهاية حلقة `for`، لذا فإن كل هذه التغييرات آمنة ومسموح بها بواسطة قواعد الاستعارة (borrowing rules).

### دوال التجزئة (Hashing Functions)

بشكل افتراضي، تستخدم `HashMap` دالة تجزئة تسمى سيبهاش (`SipHash`) يمكن أن توفر مقاومة لهجمات حجب الخدمة (denial-of-service (DoS) attacks) التي تتضمن جداول التجزئة (hash tables)[^siphash]<!-- ignore -->. هذه ليست أسرع خوارزمية تجزئة متاحة، ولكن المقايضة بين الأمان الأفضل الذي يأتي مع انخفاض الأداء تستحق العناء. إذا قمت بتحليل الكود الخاص بك ووجدت أن دالة hash الافتراضية بطيئة جدًا لأغراضك، فيمكنك التبديل إلى دالة أخرى عن طريق تحديد مجزئ (hasher) مختلف. الـ hasher هو نوع يطبق سمة `BuildHasher` (`BuildHasher trait`). سنتحدث عن traits وكيفية تطبيقها في [Chapter 10][traits]<!-- ignore -->. ليس عليك بالضرورة تطبيق hasher الخاص بك من البداية؛ يحتوي [crates.io](https://crates.io/)<!-- ignore --> على مكتبات يشاركها مستخدمو Rust الآخرون توفر hashers تطبق العديد من خوارزميات hash الشائعة.

[^siphash]: [https://en.wikipedia.org/wiki/SipHash](https://en.wikipedia.org/wiki/SipHash)

## ملخص

ستوفر vectors و strings و Hash Maps قدرًا كبيرًا من الوظائف الضرورية في البرامج عندما تحتاج إلى تخزين البيانات والوصول إليها وتعديلها. فيما يلي بعض التمارين التي يجب أن تكون الآن مجهزًا لحلها:

1. بالنظر إلى قائمة من الأعداد الصحيحة، استخدم vector وأرجع الوسيط (median) (عندما يتم فرزها، القيمة في الموضع الأوسط) والمنوال (mode) (القيمة التي تحدث في أغلب الأحيان؛ ستكون Hash Map مفيدة هنا) للقائمة.
2. قم بتحويل strings إلى اللاتينية الخنزيرية (Pig Latin). يتم نقل الحرف الساكن الأول من كل كلمة إلى نهاية الكلمة ويتم إضافة _ay_، لذا تصبح _first_ هي _irst-fay_. الكلمات التي تبدأ بحرف متحرك يتم إضافة _hay_ إلى نهايتها بدلاً من ذلك (_apple_ تصبح _apple-hay_). ضع في اعتبارك التفاصيل المتعلقة بترميز UTF-8 (`UTF-8 encoding`)!
3. باستخدام Hash Map و vectors، قم بإنشاء واجهة نصية للسماح للمستخدم بإضافة أسماء الموظفين إلى قسم في شركة؛ على سبيل المثال، "Add Sally to Engineering" أو "Add Amir to Sales". بعد ذلك، اسمح للمستخدم باسترداد قائمة بجميع الأشخاص في قسم أو جميع الأشخاص في الشركة حسب القسم، مرتبة أبجديًا.

تصف وثائق API لـ standard library الدوال التي تحتوي عليها vectors و strings و Hash Maps والتي ستكون مفيدة لهذه التمارين!

نحن ندخل في برامج أكثر تعقيدًا حيث يمكن أن تفشل العمليات، لذا فقد حان الوقت المثالي لمناقشة معالجة الأخطاء. سنفعل ذلك لاحقًا!

[validating-references-with-lifetimes]: ch10-03-lifetime-syntax.html#validating-references-with-lifetimes
[access]: #accessing-values-in-a-hash-map
[traits]: ch10-02-traits.html
