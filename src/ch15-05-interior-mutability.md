## `RefCell<T>` ونمط القابلية للتغيير الداخلية (Interior Mutability Pattern)

"القابلية للتغيير الداخلية" (Interior Mutability) هي نمط تصميم في Rust يسمح لك بتغيير البيانات حتى عندما تكون هناك مراجع غير قابلة للتغيير (Immutable References) لتلك البيانات؛ عادةً ما يكون هذا الإجراء غير مسموح به بموجب قواعد الاستعارة (Borrowing Rules). لتغيير البيانات، يستخدم هذا النمط كوداً "غير آمن" (Unsafe Code) داخل هيكل البيانات لثني قواعد Rust المعتادة التي تحكم التغيير والاستعارة. يشير الكود الـ Unsafe للمترجم (Compiler) إلى أننا نتحقق من القواعد يدوياً بدلاً من الاعتماد على الـ Compiler للتحقق منها نيابة عنا؛ سنناقش الكود الـ Unsafe بشكل أكبر في الفصل 20.

يمكننا استخدام الأنواع التي تتبع نمط الـ Interior Mutability فقط عندما نتمكن من ضمان اتباع الـ Borrowing Rules في وقت التشغيل (Runtime)، على الرغم من أن الـ Compiler لا يمكنه ضمان ذلك. يتم بعد ذلك تغليف الكود الـ Unsafe المعني بواجهة برمجية (API) آمنة، ويظل النوع الخارجي غير قابل للتغيير (Immutable).

دعنا نستكشف هذا المفهوم من خلال النظر في النوع `RefCell<T>` الذي يتبع نمط الـ Interior Mutability.

<!-- Old headings. Do not remove or links may break. -->

<a id="enforcing-borrowing-rules-at-runtime-with-refcellt"></a>

### فرض قواعد الاستعارة في وقت التشغيل (Runtime)

على عكس `Rc<T>`، يمثل النوع `RefCell<T>` ملكية فردية (Single Ownership) للبيانات التي يحملها. إذاً، ما الذي يجعل `RefCell<T>` مختلفاً عن نوع مثل `Box<T>`؟ تذكر الـ Borrowing Rules التي تعلمتها في الفصل 4:

- في أي وقت معين، يمكنك الحصول على مرجع واحد قابل للتغيير (Mutable Reference) *أو* أي عدد من الـ Immutable References (ولكن ليس كلاهما معاً).
- يجب أن تكون المراجع (References) صالحة دائماً.

مع الـ References و `Box<T>`، يتم فرض ثوابت الـ Borrowing Rules في وقت التجميع (Compile Time). أما مع `RefCell<T>`، فيتم فرض هذه الثوابت في وقت التشغيل (Runtime). مع الـ References، إذا خالفت هذه القواعد، فستحصل على خطأ من الـ Compiler. أما مع `RefCell<T>`، فإذا خالفت هذه القواعد، فسوف يهلع (Panic) برنامجك ويخرج.

تتمثل مزايا التحقق من الـ Borrowing Rules في الـ Compile Time في أنه سيتم اكتشاف الأخطاء في وقت مبكر من عملية التطوير، ولا يوجد أي تأثير على أداء الـ Runtime لأن كل التحليل يكتمل مسبقاً. لهذه الأسباب، يعد التحقق من الـ Borrowing Rules في الـ Compile Time هو الخيار الأفضل في غالبية الحالات، ولهذا السبب هو الخيار الافتراضي في Rust.

تتمثل ميزة التحقق من الـ Borrowing Rules في الـ Runtime بدلاً من ذلك في السماح ببعض سيناريوهات أمان الذاكرة (Memory-safe)، حيث كان سيتم رفضها بواسطة فحوصات الـ Compile Time. التحليل الساكن (Static Analysis)، مثل مترجم Rust، هو محافظ بطبيعته. بعض خصائص الكود مستحيلة الاكتشاف من خلال تحليل الكود: المثال الأكثر شهرة هو "مشكلة التوقف" (Halting Problem)، وهي خارج نطاق هذا الكتاب ولكنها موضوع مثير للبحث.

بسبب استحالة بعض التحليلات، إذا لم يتمكن مترجم Rust من التأكد من امتثال الكود لقواعد الملكية (Ownership Rules)، فقد يرفض برنامجاً صحيحاً؛ وبهذه الطريقة، يكون المترجم محافظاً. إذا قبلت Rust برنامجاً غير صحيح، فلن يتمكن المستخدمون من الوثوق بالضمانات التي تقدمها Rust. ومع ذلك، إذا رفضت Rust برنامجاً صحيحاً، فسوف ينزعج المبرمج، ولكن لن يحدث أي شيء كارثي. يكون النوع `RefCell<T>` مفيداً عندما تكون متأكداً من أن الكود الخاص بك يتبع الـ Borrowing Rules ولكن الـ Compiler غير قادر على فهم ذلك وضمانه.

على غرار `Rc<T>`، فإن `RefCell<T>` مخصص للاستخدام فقط في السيناريوهات أحادية المسار (Single-threaded) وسيعطيك خطأ في الـ Compile Time إذا حاولت استخدامه في سياق متعدد المسارات (Multithreaded). سنتحدث عن كيفية الحصول على وظائف `RefCell<T>` في برنامج Multithreaded في الفصل 16.

إليك ملخص لأسباب اختيار `Box<T>` أو `Rc<T>` أو `RefCell<T>`:

- يسمح `Rc<T>` بوجود مالكين متعددين لنفس البيانات؛ بينما يمتلك `Box<T>` و `RefCell<T>` مالكاً واحداً فقط.
- يسمح `Box<T>` باستعارات غير قابلة للتغيير أو قابلة للتغيير يتم فحصها في الـ Compile Time؛ ويسمح `Rc<T>` فقط بالاستعارات غير القابلة للتغيير التي يتم فحصها في الـ Compile Time؛ بينما يسمح `RefCell<T>` باستعارات غير قابلة للتغيير أو قابلة للتغيير يتم فحصها في الـ Runtime.
- لأن `RefCell<T>` يسمح باستعارات قابلة للتغيير (Mutable Borrows) يتم فحصها في الـ Runtime، يمكنك تغيير القيمة داخل `RefCell<T>` حتى عندما يكون الـ `RefCell<T>` نفسه غير قابل للتغيير.

تغيير القيمة داخل قيمة غير قابلة للتغيير هو نمط الـ Interior Mutability. دعنا ننظر في موقف تكون فيه الـ Interior Mutability مفيدة ونفحص كيف يكون ذلك ممكناً.

<!-- Old headings. Do not remove or links may break. -->

<a id="interior-mutability-a-mutable-borrow-to-an-immutable-value"></a>

### استخدام القابلية للتغيير الداخلية (Interior Mutability)

من نتائج الـ Borrowing Rules أنه عندما يكون لديك قيمة غير قابلة للتغيير، لا يمكنك استعارتها بشكل قابل للتغيير. على سبيل المثال، لن يتم تجميع هذا الكود:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/no-listing-01-cant-borrow-immutable-as-mutable/src/main.rs}}
```

إذا حاولت تجميع هذا الكود، فستحصل على الخطأ التالي:

```console
{{#include ../listings/ch15-smart-pointers/no-listing-01-cant-borrow-immutable-as-mutable/output.txt}}
```

ومع ذلك، هناك مواقف يكون من المفيد فيها أن تقوم القيمة بتغيير نفسها في دوالها المرتبطة (Methods) ولكنها تظهر غير قابلة للتغيير للكود الآخر. لن يتمكن الكود الموجود خارج الـ Methods الخاصة بالقيمة من تغيير القيمة. استخدام `RefCell<T>` هو إحدى الطرق للحصول على القدرة على امتلاك Interior Mutability، ولكن `RefCell<T>` لا يلتف على الـ Borrowing Rules تماماً: يسمح الـ Borrow Checker في الـ Compiler بهذه الـ Interior Mutability، ويتم التحقق من الـ Borrowing Rules في الـ Runtime بدلاً من ذلك. إذا انتهكت القواعد، فستحصل على `panic!` بدلاً من خطأ الـ Compiler.

دعنا نمر بمثال عملي حيث يمكننا استخدام `RefCell<T>` لتغيير قيمة غير قابلة للتغيير ونرى لماذا يعد ذلك مفيداً.

<!-- Old headings. Do not remove or links may break. -->

<a id="a-use-case-for-interior-mutability-mock-objects"></a>

#### الاختبار باستخدام الكائنات الوهمية (Mock Objects)

أحياناً أثناء الاختبار، يستخدم المبرمج نوعاً بدلاً من نوع آخر، من أجل مراقبة سلوك معين والتأكد من تنفيذه بشكل صحيح. يسمى هذا النوع البديل "بديل الاختبار" (Test Double). فكر في الأمر بمعنى "البديل السينمائي" (Stunt Double) في صناعة الأفلام، حيث يحل شخص محل الممثل للقيام بمشهد صعب بشكل خاص. تحل الـ Test Doubles محل الأنواع الأخرى عندما نقوم بتشغيل الاختبارات. "الكائنات الوهمية" (Mock Objects) هي أنواع محددة من الـ Test Doubles التي تسجل ما يحدث أثناء الاختبار حتى تتمكن من التأكد من حدوث الإجراءات الصحيحة.

لا تمتلك Rust كائنات (Objects) بنفس المعنى الموجود في اللغات الأخرى، ولا تمتلك Rust وظائف Mock Object مدمجة في المكتبة القياسية (Standard Library) كما تفعل بعض اللغات الأخرى. ومع ذلك، يمكنك بالتأكيد إنشاء هيكل (Struct) يخدم نفس أغراض الـ Mock Object.

إليك السيناريو الذي سنختبره: سننشئ مكتبة تتبع قيمة مقابل قيمة قصوى وترسل رسائل بناءً على مدى قرب القيمة الحالية من القيمة القصوى. يمكن استخدام هذه المكتبة لتتبع حصة (Quota) المستخدم لعدد استدعاءات الـ API المسموح له بإجرائها، على سبيل المثال.

ستوفر مكتبتنا فقط وظيفة تتبع مدى قرب القيمة من الحد الأقصى وما هي الرسائل التي يجب أن تكون في أي أوقات. من المتوقع أن توفر التطبيقات التي تستخدم مكتبتنا آلية إرسال الرسائل: يمكن للتطبيق إظهار الرسالة للمستخدم مباشرة، أو إرسال بريد إلكتروني، أو إرسال رسالة نصية، أو القيام بشيء آخر. لا تحتاج المكتبة إلى معرفة هذا التفصيل. كل ما تحتاجه هو شيء ينفذ سمة (Trait) سنوفرها، تسمى `Messenger`. توضح القائمة 15-20 كود المكتبة.

<Listing number="15-20" file-name="src/lib.rs" caption="مكتبة لتتبع مدى قرب القيمة من القيمة القصوى والتحذير عندما تكون القيمة عند مستويات معينة">

```rust,noplayground
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-20/src/lib.rs}}
```

</Listing>

أحد الأجزاء المهمة في هذا الكود هو أن الـ `Messenger` Trait يحتوي على Method واحدة تسمى `send` تأخذ Immutable Reference لـ `self` ونص الرسالة. هذا الـ Trait هو الواجهة التي يحتاج الـ Mock Object الخاص بنا إلى تنفيذها بحيث يمكن استخدام الـ Mock بنفس الطريقة التي يستخدم بها الكائن الحقيقي. الجزء المهم الآخر هو أننا نريد اختبار سلوك الـ `set_value` Method في الـ `LimitTracker`. يمكننا تغيير ما نمرره لمعامل `value` ولكن `set_value` لا تعيد أي شيء لكي نقوم بإجراء تأكيدات (Assertions) عليه. نريد أن نكون قادرين على القول إنه إذا أنشأنا `LimitTracker` بشيء ينفذ الـ `Messenger` Trait وقيمة معينة لـ `max` فسيتم إخبار الـ Messenger بإرسال الرسائل المناسبة عندما نمرر أرقاماً مختلفة لـ `value`.

نحتاج إلى Mock Object يقوم، بدلاً من إرسال بريد إلكتروني أو رسالة نصية عندما نستدعي `send` بتتبع الرسائل التي طُلب منه إرسالها فقط. يمكننا إنشاء نسخة جديدة من الـ Mock Object، وإنشاء `LimitTracker` يستخدم الـ Mock Object، واستدعاء الـ `set_value` Method في `LimitTracker` ثم التحقق من أن الـ Mock Object يحتوي على الرسائل التي نتوقعها. توضح القائمة 15-21 محاولة لتنفيذ Mock Object للقيام بذلك، لكن الـ Borrow Checker لن يسمح بذلك.

<Listing number="15-21" file-name="src/lib.rs" caption="محاولة لتنفيذ MockMessenger غير مسموح بها من قبل مدقق الاستعارة">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-21/src/lib.rs:here}}
```

</Listing>

يعرف كود الاختبار هذا Struct باسم `MockMessenger` يحتوي على حقل `sent_messages` مع `Vec` من قيم `String` لتتبع الرسائل التي طُلب منه إرسالها. كما نعرف دالة مرتبطة `new` لجعل إنشاء قيم `MockMessenger` جديدة تبدأ بقائمة فارغة من الرسائل أمراً مريحاً. ثم نقوم بتنفيذ الـ `Messenger` Trait لـ `MockMessenger` حتى نتمكن من إعطاء `MockMessenger` لـ `LimitTracker`. في تعريف الـ `send` Method، نأخذ الرسالة الممررة كمعامل ونخزنها في قائمة `sent_messages` الخاصة بالـ `MockMessenger`.

في الاختبار، نختبر ما يحدث عندما يُطلب من الـ `LimitTracker` تعيين `value` لشيء يمثل أكثر من 75 بالمائة من قيمة `max`. أولاً، ننشئ `MockMessenger` جديداً، والذي سيبدأ بقائمة فارغة من الرسائل. ثم ننشئ `LimitTracker` جديداً ونعطيه مرجعاً للـ `MockMessenger` الجديد وقيمة `max` قدرها `100`. نستدعي الـ `set_value` Method في الـ `LimitTracker` بقيمة `80` وهي أكثر من 75 بالمائة من 100. ثم نؤكد أن قائمة الرسائل التي يتتبعها الـ `MockMessenger` يجب أن تحتوي الآن على رسالة واحدة.

ومع ذلك، هناك مشكلة واحدة في هذا الاختبار، كما هو موضح هنا:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-21/output.txt}}
```

لا يمكننا تعديل الـ `MockMessenger` لتتبع الرسائل، لأن الـ `send` Method تأخذ Immutable Reference لـ `self`. لا يمكننا أيضاً أخذ الاقتراح من نص الخطأ باستخدام `&mut self` في كل من الـ `impl` Method وتعريف الـ Trait. نحن لا نريد تغيير الـ `Messenger` Trait لمجرد مصلحة الاختبار. بدلاً من ذلك، نحتاج إلى إيجاد طريقة لجعل كود الاختبار الخاص بنا يعمل بشكل صحيح مع تصميمنا الحالي.

هذا موقف يمكن أن تساعد فيه الـ Interior Mutability! سنقوم بتخزين الـ `sent_messages` داخل `RefCell<T>` وبعد ذلك ستتمكن الـ `send` Method من تعديل `sent_messages` لتخزين الرسائل التي رأيناها. توضح القائمة 15-22 كيف يبدو ذلك.

<Listing number="15-22" file-name="src/lib.rs" caption="استخدام RefCell<T> لتغيير قيمة داخلية بينما تعتبر القيمة الخارجية غير قابلة للتغيير">

```rust,noplayground
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-22/src/lib.rs:here}}
```

</Listing>

أصبح حقل `sent_messages` الآن من النوع `RefCell<Vec<String>>` بدلاً من `Vec<String>`. في دالة `new` ننشئ نسخة `RefCell<Vec<String>>` جديدة حول الـ Vector الفارغ.

بالنسبة لتنفيذ الـ `send` Method، لا يزال المعامل الأول عبارة عن استعارة غير قابلة للتغيير (Immutable Borrow) لـ `self` وهو ما يطابق تعريف الـ Trait. نستدعي `borrow_mut` على الـ `RefCell<Vec<String>>` في `self.sent_messages` للحصول على مرجع قابل للتغيير (Mutable Reference) للقيمة الموجودة داخل الـ `RefCell<Vec<String>>` وهي الـ Vector. بعد ذلك، يمكننا استدعاء `push` على الـ Mutable Reference للـ Vector لتتبع الرسائل المرسلة أثناء الاختبار.

التغيير الأخير الذي يتعين علينا إجراؤه هو في الـ Assertion: لمعرفة عدد العناصر الموجودة في الـ Vector الداخلي، نستدعي `borrow` على الـ `RefCell<Vec<String>>` للحصول على Immutable Reference للـ Vector.

الآن بعد أن رأيت كيفية استخدام `RefCell<T>` دعنا نتعمق في كيفية عمله!

<!-- Old headings. Do not remove or links may break. -->

<a id="keeping-track-of-borrows-at-runtime-with-refcellt"></a>

#### تتبع الاستعارات في وقت التشغيل (Runtime)

عند إنشاء Immutable References و Mutable References، نستخدم صيغة `&` و `&mut` على التوالي. مع `RefCell<T>` نستخدم دالتي `borrow` و `borrow_mut` وهما جزء من الـ API الآمن الذي ينتمي لـ `RefCell<T>`. تعيد دالة `borrow` نوع المؤشر الذكي (Smart Pointer) المسمى `Ref<T>` وتعيد `borrow_mut` نوع الـ Smart Pointer المسمى `RefMut<T>`. ينفذ كلا النوعين سمة `Deref` لذا يمكننا معاملتهما مثل الـ References العادية.

يتتبع `RefCell<T>` عدد الـ Smart Pointers من نوع `Ref<T>` و `RefMut<T>` النشطة حالياً. في كل مرة نستدعي فيها `borrow` يزيد `RefCell<T>` من عداد الاستعارات غير القابلة للتغيير النشطة. عندما تخرج قيمة `Ref<T>` عن الـ Scope، ينخفض عداد الـ Immutable Borrows بمقدار 1. تماماً مثل الـ Borrowing Rules في الـ Compile Time، يسمح لنا `RefCell<T>` بامتلاك العديد من الـ Immutable Borrows أو Mutable Borrow واحد في أي نقطة زمنية.

إذا حاولنا انتهاك هذه القواعد، فبدلاً من الحصول على خطأ من الـ Compiler كما هو الحال مع الـ References، فإن تنفيذ `RefCell<T>` سوف يهلع (Panic) في الـ Runtime. توضح القائمة 15-23 تعديلاً لتنفيذ `send` في القائمة 15-22. نحن نحاول عمداً إنشاء استعارتين قابلتين للتغيير نشطتين لنفس الـ Scope لتوضيح أن `RefCell<T>` يمنعنا من القيام بذلك في الـ Runtime.

<Listing number="15-23" file-name="src/lib.rs" caption="إنشاء مرجعين قابلين للتغيير في نفس النطاق لرؤية أن RefCell<T> سوف يهلع">

```rust,ignore,panics
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-23/src/lib.rs:here}}
```

</Listing>

ننشئ متغيراً `one_borrow` للـ `RefMut<T>` Smart Pointer المرتجع من `borrow_mut`. ثم ننشئ Mutable Borrow آخر بنفس الطريقة في المتغير `two_borrow`. هذا يؤدي لإنشاء مرجعين قابلين للتغيير في نفس الـ Scope، وهو أمر غير مسموح به. عندما نقوم بتشغيل الاختبارات لمكتبتنا، سيتم تجميع الكود في القائمة 15-23 دون أي أخطاء، لكن الاختبار سيفشل:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-23/output.txt}}
```

لاحظ أن الكود هلع مع الرسالة `already borrowed: BorrowMutError`. هذه هي الطريقة التي يتعامل بها `RefCell<T>` مع انتهاكات الـ Borrowing Rules في الـ Runtime.

إن اختيار اكتشاف أخطاء الاستعارة في الـ Runtime بدلاً من الـ Compile Time، كما فعلنا هنا، يعني أنك قد تجد أخطاء في الكود الخاص بك في وقت لاحق من عملية التطوير: ربما ليس حتى يتم نشر الكود الخاص بك في بيئة الإنتاج (Production). أيضاً، سيتحمل الكود الخاص بك عقوبة أداء بسيطة في الـ Runtime نتيجة لتتبع الاستعارات في الـ Runtime بدلاً من الـ Compile Time. ومع ذلك، فإن استخدام `RefCell<T>` يجعل من الممكن كتابة Mock Object يمكنه تعديل نفسه لتتبع الرسائل التي رآها أثناء استخدامه في سياق لا يُسمح فيه إلا بالقيم غير القابلة للتغيير. يمكنك استخدام `RefCell<T>` على الرغم من مقايضاته للحصول على وظائف أكثر مما توفره الـ References العادية.

<!-- Old headings. Do not remove or links may break. -->

<a id="having-multiple-owners-of-mutable-data-by-combining-rc-t-and-ref-cell-t"></a>
<a id="allowing-multiple-owners-of-mutable-data-with-rct-and-refcellt"></a>

### السماح بمالكين متعددين للبيانات القابلة للتغيير

الطريقة الشائعة لاستخدام `RefCell<T>` هي بالاشتراك مع `Rc<T>`. تذكر أن `Rc<T>` يسمح لك بامتلاك مالكين متعددين لبعض البيانات، ولكنه يمنح فقط وصولاً غير قابل للتغيير لتلك البيانات. إذا كان لديك `Rc<T>` يحمل `RefCell<T>` فيمكنك الحصول على قيمة يمكن أن يكون لها مالكون متعددون *و* يمكنك تغييرها!

على سبيل المثال، تذكر مثال قائمة الـ Cons في القائمة 15-18 حيث استخدمنا `Rc<T>` للسماح لقوائم متعددة بمشاركة ملكية قائمة أخرى. لأن `Rc<T>` يحمل فقط قيماً غير قابلة للتغيير، لا يمكننا تغيير أي من القيم في القائمة بمجرد إنشائها. دعنا نضيف `RefCell<T>` لقدرته على تغيير القيم في القوائم. توضح القائمة 15-24 أنه باستخدام `RefCell<T>` في تعريف الـ `Cons` يمكننا تعديل القيمة المخزنة في جميع القوائم.

<Listing number="15-24" file-name="src/main.rs" caption="استخدام Rc<RefCell<i32>> لإنشاء List يمكننا تغييرها">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-24/src/main.rs}}
```

</Listing>

ننشئ قيمة هي نسخة من `Rc<RefCell<i32>>` ونخزنها في متغير باسم `value` حتى نتمكن من الوصول إليها مباشرة لاحقاً. ثم ننشئ `List` في `a` مع تنويعة `Cons` تحمل `value`. نحتاج إلى استنساخ (Clone) لـ `value` بحيث يمتلك كل من `a` و `value` ملكية القيمة الداخلية `5` بدلاً من نقل الملكية من `value` إلى `a` أو جعل `a` يستعير من `value`.

نغلف القائمة `a` في `Rc<T>` بحيث عندما ننشئ القائمتين `b` و `c` يمكنهما الإشارة إلى `a` وهو ما فعلناه في القائمة 15-18.

بعد أن أنشأنا القوائم في `a` و `b` و `c` نريد إضافة 10 إلى القيمة الموجودة في `value`. نقوم بذلك عن طريق استدعاء `borrow_mut` على `value` والذي يستخدم ميزة إلغاء الإسناد التلقائي (Automatic Dereferencing) التي ناقشناها في قسم ["أين هو عامل `->`؟"][wheres-the---operator] في الفصل 5 لإلغاء إسناد الـ `Rc<T>` إلى قيمة الـ `RefCell<T>` الداخلية. تعيد دالة `borrow_mut` مؤشراً ذكياً من نوع `RefMut<T>` ونستخدم عامل إلغاء الإسناد (Dereference Operator) عليه ونغير القيمة الداخلية.

عندما نطبع `a` و `b` و `c` يمكننا أن نرى أنها جميعاً تحتوي على القيمة المعدلة `15` بدلاً من `5`:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-24/output.txt}}
```

هذه التقنية رائعة جداً! باستخدام `RefCell<T>` لدينا قيمة `List` تبدو من الخارج غير قابلة للتغيير. ولكن يمكننا استخدام الـ Methods في `RefCell<T>` التي توفر وصولاً إلى الـ Interior Mutability الخاصة بها حتى نتمكن من تعديل بياناتنا عندما نحتاج إلى ذلك. تحمينا فحوصات الـ Runtime للـ Borrowing Rules من "تسابق البيانات" (Data Races)، وأحياناً يستحق الأمر التضحية ببعض السرعة مقابل هذه المرونة في هياكل بياناتنا. لاحظ أن `RefCell<T>` لا يعمل مع الكود متعدد المسارات (Multithreaded)! الـ `Mutex<T>` هو النسخة الآمنة للمسارات (Thread-safe) من `RefCell<T>` وسنناقش `Mutex<T>` في الفصل 16.

[wheres-the---operator]: ch05-03-method-syntax.html#wheres-the---operator
