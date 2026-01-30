## تحسين مشروع الإدخال والإخراج الخاص بنا (Improving Our I/O Project)

باستخدام هذه المعرفة الجديدة حول المكررات (Iterators)، يمكننا تحسين مشروع الإدخال والإخراج (I/O) في الفصل 12 باستخدام Iterators لجعل أجزاء الكود (Code) أكثر وضوحاً وإيجازاً. دعنا نلقي نظرة على كيفية قيام Iterators بتحسين تنفيذنا لدالة (Function) `Config::build` ودالة `search`.

### إزالة `clone` باستخدام مكرر (Iterator)

في القائمة (Listing) 12-6، أضفنا Code يأخذ شريحة (Slice) من قيم `String` وأنشأ مثيلاً (Instance) من هيكل (Struct) `Config` عن طريق الفهرسة (Indexing) في Slice واستنساخ (Cloning) القيم، مما يسمح لـ Struct `Config` بامتلاك تلك القيم. في Listing 13-17، أعدنا إنتاج تنفيذ Function `Config::build` كما كان في Listing 12-23.

<Listing number="13-17" file-name="src/main.rs" caption="إعادة إنتاج دالة `Config::build` من القائمة 12-23">

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-12-23-reproduced/src/main.rs:ch13}}
```

</Listing>

في ذلك الوقت، قلنا ألا تقلق بشأن استدعاءات `clone` غير الفعالة لأننا سنزيلها في المستقبل. حسناً، لقد حان ذلك الوقت الآن!

احتجنا إلى `clone` هنا لأن لدينا Slice بعناصر `String` في الوسيط (Parameter) `args` ، لكن Function `build` لا تمتلك `args`. لإعادة ملكية (Ownership) Instance `Config` ، كان علينا استنساخ القيم من حقول (Fields) `query` و `file_path` الخاصة بـ `Config` بحيث يمكن لـ Instance `Config` امتلاك قيمه الخاصة.

مع معرفتنا الجديدة حول Iterators، يمكننا تغيير Function `build` لتأخذ ملكية Iterator كـ Argument بدلاً من استعارة (Borrowing) Slice. سنستخدم وظائف Iterator بدلاً من Code الذي يتحقق من طول Slice ويقوم بـ Indexing في مواقع محددة. سيؤدي هذا إلى توضيح ما تفعله Function `Config::build` لأن Iterator سيصل إلى القيم.

بمجرد أن تأخذ `Config::build` ملكية Iterator وتتوقف عن استخدام عمليات Indexing التي تقوم بـ Borrow، يمكننا نقل (Move) قيم `String` من Iterator إلى `Config` بدلاً من استدعاء `clone` وإجراء تخصيص (Allocation) جديد.

#### استخدام المكرر المُعاد مباشرة

افتح ملف _src/main.rs_ الخاص بمشروع I/O، والذي يجب أن يبدو كالتالي:

<span class="filename">اسم الملف: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-12-24-reproduced/src/main.rs:ch13}}
```

سنقوم أولاً بتغيير بداية Function `main` التي كانت لدينا في Listing 12-24 إلى Code الموجود في Listing 13-18، والذي يستخدم هذه المرة Iterator. لن يتم تحويل هذا برمجياً (Compile) حتى نقوم بتحديث `Config::build` أيضاً.

<Listing number="13-18" file-name="src/main.rs" caption="تمرير القيمة المعادة من `env::args` إلى `Config::build` ">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-18/src/main.rs:here}}
```

</Listing>

تعيد دالة `env::args` مكرراً (Iterator)! وبدلاً من تجميع قيم Iterator في متجه (Vector) ثم تمرير Slice إلى `Config::build` ، فإننا الآن نمرر ملكية Iterator المُعاد من `env::args` إلى `Config::build` مباشرة.

بعد ذلك، نحتاج إلى تحديث تعريف `Config::build`. دعنا نغير توقيع (Signature) `Config::build` ليبدو مثل Listing 13-19. هذا لا يزال لن يتم Compile ، لأننا بحاجة إلى تحديث جسم الدالة (Function Body).

<Listing number="13-19" file-name="src/main.rs" caption="تحديث توقيع `Config::build` ليتوقع مكرراً">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-19/src/main.rs:here}}
```

</Listing>

توضح وثائق المكتبة القياسية (Standard Library Documentation) لدالة `env::args` أن نوع Iterator الذي تعيده هو `std::env::Args` ، وهذا النوع ينفذ سمة (Trait) الـ `Iterator` ويعيد قيم `String`.

لقد قمنا بتحديث Signature لـ Function `Config::build` بحيث يكون لـ Parameter `args` نوع عام (Generic Type) مع قيود السمة (Trait Bounds) `impl Iterator<Item = String>` بدلاً من `&[String]`. هذا الاستخدام لصيغة `impl Trait` التي ناقشناها في قسم ["استخدام السمات كـ Parameters"][impl-trait]<!-- ignore --> من الفصل 10 يعني أن `args` يمكن أن يكون أي نوع ينفذ Trait `Iterator` ويعيد عناصر `String`.

لأننا نأخذ ملكية `args` وسنقوم بتعديل (Mutating) `args` عن طريق التكرار عليه، يمكننا إضافة الكلمة المفتاحية `mut` في مواصفات Parameter `args` لجعله قابلاً للتغير (Mutable).

<!-- العناوين القديمة. لا تقم بإزالتها وإلا قد تتعطل الروابط. -->

<a id="using-iterator-trait-methods-instead-of-indexing"></a>

#### استخدام طرق سمة `Iterator`

بعد ذلك، سنقوم بإصلاح Function Body لـ `Config::build`. نظراً لأن `args` ينفذ Trait `Iterator` ، فإننا نعلم أنه يمكننا استدعاء طريقة (Method) `next` عليه! يقوم Listing 13-20 بتحديث Code من Listing 12-23 لاستخدام Method `next`.

<Listing number="13-20" file-name="src/main.rs" caption="تغيير جسم `Config::build` لاستخدام طرق المكرر">

```rust,ignore,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-20/src/main.rs:here}}
```

</Listing>

تذكر أن القيمة الأولى في القيمة المعادة من `env::args` هي اسم البرنامج. نريد تجاهل ذلك والوصول إلى القيمة التالية، لذا نستدعي أولاً `next` ولا نفعل شيئاً بالقيمة المعادة. بعد ذلك، نستدعي `next` للحصول على القيمة التي نريد وضعها في Field `query` الخاص بـ `Config`. إذا أعاد `next` القيمة `Some` ، فإننا نستخدم `match` لاستخراج القيمة. وإذا أعاد `None` ، فهذا يعني أنه لم يتم توفير وسائط كافية، ونقوم بالإرجاع مبكراً مع قيمة `Err`. نفعل الشيء نفسه لقيمة `file_path`.

<!-- العناوين القديمة. لا تقم بإزالتها وإلا قد تتعطل الروابط. -->

<a id="making-code-clearer-with-iterator-adapters"></a>

### توضيح الكود باستخدام محولات المكرر (Iterator Adapters)

يمكننا أيضاً الاستفادة من Iterators في Function `search` في مشروع I/O الخاص بنا، والتي أُعيد إنتاجها هنا في Listing 13-21 كما كانت في Listing 12-19.

<Listing number="13-21" file-name="src/lib.rs" caption="تنفيذ دالة `search` من القائمة 12-19">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-19/src/lib.rs:ch13}}
```

</Listing>

يمكننا كتابة هذا Code بطريقة أكثر إيجازاً باستخدام طرق محولات المكرر (Iterator Adapter Methods). القيام بذلك يجنبنا أيضاً وجود Vector وسيط قابل للتغير لـ `results`. يفضل أسلوب البرمجة الوظيفية (Functional Programming) تقليل كمية الحالة القابلة للتغير (Mutable State) لجعل Code أكثر وضوحاً. قد تتيح إزالة Mutable State تحسيناً مستقبلياً لجعل البحث يحدث بالتوازي لأننا لن نضطر إلى إدارة الوصول المتزامن (Concurrent Access) إلى Vector الـ `results`. يوضح Listing 13-22 هذا التغيير.

<Listing number="13-22" file-name="src/lib.rs" caption="استخدام طرق محول المكرر في تنفيذ دالة `search` ">

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-22/src/lib.rs:here}}
```

</Listing>

تذكر أن الغرض من Function `search` هو إرجاع جميع الأسطر في `contents` التي تحتوي على `query`. على غرار مثال الـ `filter` في Listing 13-16، يستخدم هذا Code محول `filter` للاحتفاظ فقط بالأسطر التي تعيد لها `line.contains(query)` القيمة `true`. ثم نقوم بتجميع (Collect) الأسطر المتطابقة في Vector آخر باستخدام `collect`. أبسط بكثير! لا تتردد في إجراء نفس التغيير لاستخدام طرق Iterator في Function `search_case_insensitive` أيضاً.

لمزيد من التحسين، قم بإرجاع Iterator من Function `search` عن طريق إزالة استدعاء `collect` وتغيير نوع الإرجاع إلى `impl Iterator<Item = &'a str>` بحيث تصبح Function عبارة عن Iterator Adapter. لاحظ أنك ستحتاج أيضاً إلى تحديث الاختبارات! ابحث في ملف كبير باستخدام أداة `minigrep` الخاصة بك قبل وبعد إجراء هذا التغيير لملاحظة الفرق في السلوك. قبل هذا التغيير، لن يطبع البرنامج أي نتائج حتى يجمع كل النتائج، ولكن بعد التغيير، ستُطبع النتائج فور العثور على كل سطر مطابق لأن حلقة (Loop) الـ `for` في Function `run` قادرة على الاستفادة من الكسل (Laziness) الخاص بـ Iterator.

<!-- العناوين القديمة. لا تقم بإزالتها وإلا قد تتعطل الروابط. -->

<a id="choosing-between-loops-or-iterators"></a>

### الاختيار بين الحلقات أو المكررات (Loops or Iterators)

السؤال المنطقي التالي هو أي أسلوب يجب أن تختاره في Code الخاص بك ولماذا: التنفيذ الأصلي في Listing 13-21 أو الإصدار الذي يستخدم Iterators في Listing 13-22 (بافتراض أننا نجمع كل النتائج قبل إرجاعها بدلاً من إرجاع Iterator). يفضل معظم مبرمجي Rust استخدام أسلوب Iterator. من الصعب قليلاً التعود عليه في البداية، ولكن بمجرد أن تعتاد على مختلف Iterator Adapters وما تفعله، يمكن أن تكون Iterators أسهل في الفهم. بدلاً من العبث بأجزاء Loops المختلفة وبناء Vectors جديدة، يركز Code على الهدف عالي المستوى لـ Loop. هذا يجرد بعض Code الشائع بحيث يسهل رؤية المفاهيم الفريدة لهذا Code، مثل شرط التصفية (Filtering Condition) الذي يجب أن يمر به كل عنصر في Iterator.

ولكن هل التنفيذان متكافئان حقاً؟ قد يكون الافتراض البديهي هو أن Loop منخفض المستوى سيكون أسرع. دعنا نتحدث عن الأداء (Performance).

[impl-trait]: ch10-02-traits.html#traits-as-parameters
