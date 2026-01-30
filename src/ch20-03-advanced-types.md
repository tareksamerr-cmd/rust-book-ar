## أنواع متقدمة (Advanced Types)

يحتوي نظام الأنواع (type system) في Rust على بعض الميزات التي ذكرناها سابقاً ولكن لم نناقشها بعد. سنبدأ بمناقشة الأنواع الجديدة (newtypes) بشكل عام بينما نفحص سبب فائدتها كأنواع. بعد ذلك، سننتقل إلى الأسماء المستعارة للأنواع (type aliases)، وهي ميزة مشابهة لـ newtypes ولكن بدلالات (semantics) مختلفة قليلاً. سنناقش أيضاً نوع الـ `!` والأنواع ذات الحجم الديناميكي (dynamically sized types).

<!-- Old headings. Do not remove or links may break. -->

<a id="using-the-newtype-pattern-for-type-safety-and-abstraction"></a>

### سلامة الأنواع والتجريد باستخدام نمط الـ Newtype (Type Safety and Abstraction with the Newtype Pattern)

يفترض هذا القسم أنك قرأت القسم السابق ["تطبيق السمات الخارجية باستخدام نمط الـ Newtype"][newtype]. نمط الـ newtype مفيد أيضاً لمهام تتجاوز تلك التي ناقشناها حتى الآن، بما في ذلك فرض أن القيم لا يتم الخلط بينها أبداً بشكل ثابت (statically) والإشارة إلى وحدات القيمة. لقد رأيت مثالاً على استخدام newtypes للإشارة إلى الوحدات في القائمة 20-16: تذكر أن هياكل (structs) الـ `Millimeters` و `Meters` غلفت قيم `u32` في newtype. إذا كتبنا دالة (function) بمعامل (parameter) من نوع `Millimeters` ، فلن نتمكن من تجميع برنامج حاول بالخطأ استدعاء تلك الـ function بقيمة من نوع `Meters` أو `u32` عادي.

يمكننا أيضاً استخدام نمط الـ newtype لتجريد (abstract away) بعض تفاصيل التطبيق (implementation details) لنوع ما: يمكن للنوع الجديد كشف واجهة برمجة تطبيقات عامة (public API) تختلف عن الـ API للنوع الداخلي الخاص (private inner type).

يمكن لـ newtypes أيضاً إخفاء التطبيق الداخلي. على سبيل المثال، يمكننا توفير نوع `People` لتغليف `HashMap<i32, String>` يخزن معرف (ID) الشخص المرتبط باسمه. الكود الذي يستخدم `People` سيتفاعل فقط مع الـ public API الذي نوفره، مثل method لإضافة سلسلة نصية للاسم إلى مجموعة `People` ؛ لن يحتاج هذا الكود إلى معرفة أننا نخصص ID من نوع `i32` للأسماء داخلياً. نمط الـ newtype هو طريقة خفيفة لتحقيق التغليف (encapsulation) لإخفاء implementation details ، والتي ناقشناها في قسم ["التغليف الذي يخفي تفاصيل التطبيق"][encapsulation-that-hides-implementation-details] في الفصل الثامن عشر.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-type-synonyms-with-type-aliases"></a>

### مرادفات الأنواع والأسماء المستعارة للأنواع (Type Synonyms and Type Aliases)

توفر Rust القدرة على التصريح عن _اسم مستعار للنوع_ (type alias) لإعطاء نوع موجود اسماً آخر. لهذا نستخدم الكلمة المفتاحية `type`. على سبيل المثال، يمكننا إنشاء الـ alias المسمى `Kilometers` للنوع `i32` هكذا:

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-04-kilometers-alias/src/main.rs:here}}
```

الآن الـ alias المسمى `Kilometers` هو _مرادف_ (synonym) لـ `i32` ؛ على عكس نوعي `Millimeters` و `Meters` اللذين أنشأناهما في القائمة 20-16، فإن `Kilometers` ليس نوعاً جديداً ومنفصلاً. سيتم التعامل مع القيم التي لها النوع `Kilometers` بنفس طريقة التعامل مع قيم النوع `i32`:

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-04-kilometers-alias/src/main.rs:there}}
```

لأن `Kilometers` و `i32` هما نفس النوع، يمكننا إضافة قيم من كلا النوعين ويمكننا تمرير قيم `Kilometers` إلى functions تأخذ parameters من نوع `i32`. ومع ذلك، باستخدام هذه الطريقة، لا نحصل على فوائد فحص الأنواع (type-checking) التي نحصل عليها من نمط الـ newtype الذي ناقشناه سابقاً. بمعنى آخر، إذا خلطنا بين قيم `Kilometers` و `i32` في مكان ما، فلن يعطينا المترجم (compiler) خطأً.

حالة الاستخدام الرئيسية لـ type synonyms هي تقليل التكرار. على سبيل المثال، قد يكون لدينا نوع طويل مثل هذا:

```rust,ignore
Box<dyn Fn() + Send + 'static>
```

كتابة هذا النوع الطويل في تواقيع الدوال (function signatures) وكـ annotations للأنواع في كل مكان في الكود يمكن أن يكون متعباً وعرضة للخطأ. تخيل وجود مشروع مليء بكود مثل ذلك الموجود في القائمة 20-25.

<Listing number="20-25" caption="Using a long type in many places">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-25/src/main.rs:here}}
```

</Listing>

يجعل الـ type alias هذا الكود أكثر قابلية للإدارة عن طريق تقليل التكرار. في القائمة 20-26، قدمنا alias باسم `Thunk` للنوع المطول ويمكننا استبدال جميع استخدامات النوع بالـ alias الأقصر `Thunk`.

<Listing number="20-26" caption="Introducing a type alias, `Thunk`, to reduce repetition">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-26/src/main.rs:here}}
```

</Listing>

هذا الكود أسهل بكثير في القراءة والكتابة! اختيار اسم ذو معنى لـ type alias يمكن أن يساعد في إيصال نيتك (intent) أيضاً (_thunk_ هي كلمة لكود سيتم تقييمه في وقت لاحق، لذا فهو اسم مناسب لـ closure يتم تخزينه).

تُستخدم type aliases أيضاً بشكل شائع مع نوع `Result<T, E>` لتقليل التكرار. فكر في وحدة (module) الـ `std::io` في المكتبة القياسية (standard library). غالباً ما تعيد عمليات الإدخال/الإخراج (I/O) قيمة `Result<T, E>` للتعامل مع المواقف التي تفشل فيها العمليات. تحتوي هذه المكتبة على struct باسم `std::io::Error` يمثل جميع أخطاء I/O الممكنة. العديد من الـ functions في `std::io` ستعيد `Result<T, E>` حيث الـ `E` هي `std::io::Error` ، مثل هذه الـ functions في سمة (trait) الـ `Write`:

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-05-write-trait/src/lib.rs}}
```

يتكرر `Result<..., Error>` كثيراً. على هذا النحو، لدى `std::io` هذا التصريح عن type alias:

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-06-result-alias/src/lib.rs:here}}
```

لأن هذا التصريح موجود في module الـ `std::io` ، يمكننا استخدام الـ alias المؤهل بالكامل `std::io::Result<T>` ؛ أي `Result<T, E>` مع ملء `E` كـ `std::io::Error`. تنتهي function signatures لـ trait الـ `Write` لتبدو هكذا:

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-06-result-alias/src/lib.rs:there}}
```

يساعد الـ type alias بطريقتين: يجعل الكود أسهل في الكتابة _و_ يعطينا واجهة (interface) متسقة عبر كل `std::io`. ولأنه alias، فهو مجرد `Result<T, E>` آخر، مما يعني أنه يمكننا استخدام أي methods تعمل على `Result<T, E>` معه، بالإضافة إلى syntax خاص مثل عامل الـ `?`.

### نوع الـ Never الذي لا يعود أبداً (The Never Type That Never Returns)

لدى Rust نوع خاص باسم `!` يُعرف في لغة نظرية الأنواع بـ _النوع الفارغ_ (empty type) لأنه لا يحتوي على قيم. نحن نفضل تسميته _نوع الـ never_ (never type) لأنه يقف في مكان نوع الإرجاع عندما لا تعود الـ function أبداً. إليك مثال:

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-07-never-type/src/lib.rs:here}}
```

يُقرأ هذا الكود على أن "الـ function المسمى `bar` لا تعود أبداً". الـ functions التي لا تعود أبداً تسمى _دوال متباعدة_ (diverging functions). لا يمكننا إنشاء قيم من النوع `!` ، لذا لا يمكن لـ `bar` أن تعود أبداً.

ولكن ما فائدة نوع لا يمكنك أبداً إنشاء قيم له؟ تذكر الكود من القائمة 2-5، وهو جزء من لعبة تخمين الأرقام؛ لقد أعدنا إنتاج جزء منه هنا في القائمة 20-27.

<Listing number="20-27" caption="A `match` with an arm that ends in `continue`">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-05/src/main.rs:ch19}}
```

</Listing>

في ذلك الوقت، تخطينا بعض التفاصيل في هذا الكود. في قسم ["بنية التحكم في التدفق `match`"][the-match-control-flow-construct] في الفصل السادس، ناقشنا أن أذرع (arms) الـ `match` يجب أن تعيد جميعها نفس النوع. لذا، على سبيل المثال، الكود التالي لا يعمل:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-08-match-arms-different-types/src/main.rs:here}}
```

يجب أن يكون نوع `guess` في هذا الكود integer _و_ string، وتتطلب Rust أن يكون لـ `guess` نوع واحد فقط. لذا، ماذا يعيد `continue` ؟ كيف سُمح لنا بإرجاع `u32` من ذراع واحد وامتلاك ذراع آخر ينتهي بـ `continue` في القائمة 20-27؟

كما قد خمنت، لـ `continue` قيمة `!` . أي عندما تحسب Rust نوع `guess` ، فإنها تنظر في كلا ذراعي match، الأول بقيمة `u32` والثاني بقيمة `!` . ولأن `!` لا يمكن أن يكون له قيمة أبداً، تقرر Rust أن نوع `guess` هو `u32`.
```
الطريقة الرسمية لوصف هذا السلوك هي أن التعبيرات من النوع `!` يمكن إجبارها (coerced) على أي نوع آخر. يُسمح لنا بإنهاء ذراع الـ `match` هذا بـ `continue` لأن `continue` لا يعيد قيمة؛ بدلاً من ذلك، فإنه ينقل التحكم مرة أخرى إلى أعلى الحلقة (loop)، لذا في حالة الـ `Err` ، لا نقوم أبداً بتعيين قيمة لـ `guess`.

نوع الـ never مفيد مع ماكرو (macro) الـ `panic!` أيضاً. تذكر دالة الـ `unwrap` التي نستدعيها على قيم `Option<T>` لإنتاج قيمة أو الذعر (panic) بهذا التعريف:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-09-unwrap-definition/src/lib.rs:here}}
```

في هذا الكود، يحدث نفس الشيء كما في الـ `match` في القائمة 20-27: ترى Rust أن `val` لها النوع `T` و `panic!` لها النوع `!` ، لذا فإن نتيجة تعبير الـ `match` الإجمالي هي `T`. يعمل هذا الكود لأن `panic!` لا ينتج قيمة؛ بل ينهي البرنامج. في حالة الـ `None` ، لن نعيد قيمة من `unwrap` ، لذا فإن هذا الكود صالح.

تعبير أخير له النوع `!` هو الحلقة (loop):

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-10-loop-returns-never/src/main.rs:here}}
```

هنا، الـ loop لا تنتهي أبداً، لذا `!` هي قيمة التعبير. ومع ذلك، لن يكون هذا صحيحاً إذا قمنا بتضمين `break` ، لأن الـ loop ستنتهي عندما تصل إلى الـ `break`.

### الأنواع ذات الحجم الديناميكي وسمة الـ Sized (Dynamically Sized Types and the Sized Trait)

تحتاج Rust إلى معرفة تفاصيل معينة حول أنواعها، مثل مقدار المساحة التي يجب تخصيصها لقيمة من نوع معين. هذا يترك جانباً واحداً من نظام الأنواع الخاص بها مربكاً قليلاً في البداية: مفهوم _الأنواع ذات الحجم الديناميكي_ (dynamically sized types). يُشار إليها أحياناً باسم _DSTs_ أو _الأنواع غير محددة الحجم_ (unsized types)، وتسمح لنا هذه الأنواع بكتابة كود باستخدام قيم لا يمكننا معرفة حجمها إلا في وقت التشغيل (runtime).

دعونا نتعمق في تفاصيل نوع ذو حجم ديناميكي يسمى `str` ، والذي كنا نستخدمه طوال الكتاب. هذا صحيح، ليس `&str` ، بل `str` بمفرده هو DST. في كثير من الحالات، مثل عند تخزين نص أدخله مستخدم، لا يمكننا معرفة طول السلسلة النصية حتى runtime. هذا يعني أنه لا يمكننا إنشاء متغير من نوع `str` ، ولا يمكننا أخذ وسيط (argument) من نوع `str`. فكر في الكود التالي، الذي لا يعمل:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-11-cant-create-str/src/main.rs:here}}
```

تحتاج Rust إلى معرفة مقدار الذاكرة التي يجب تخصيصها لأي قيمة من نوع معين، ويجب أن تستخدم جميع قيم النوع نفس مقدار الذاكرة. إذا سمحت لنا Rust بكتابة هذا الكود، فستحتاج قيمتا `str` هاتان إلى شغل نفس مقدار المساحة. لكن لهما أطوال مختلفة: `s1` يحتاج إلى 12 بايت من التخزين و `s2` يحتاج إلى 15. وهذا هو السبب في أنه ليس من الممكن إنشاء متغير يحمل نوعاً ذا حجم ديناميكي.

لذا، ماذا نفعل؟ في هذه الحالة، أنت تعرف الإجابة بالفعل: نجعل نوع `s1` و `s2` شريحة سلسلة نصية (string slice) (`&str`) بدلاً من `str`. تذكر من قسم ["شرائح السلسلة النصية"][string-slices] في الفصل الرابع أن هيكل بيانات الشريحة (slice data structure) يخزن فقط موضع البداية وطول الشريحة. لذا، على الرغم من أن `&T` هي قيمة واحدة تخزن عنوان الذاكرة حيث يوجد `T` ، فإن string slice هي _قيمتان_: عنوان الـ `str` وطوله. على هذا النحو، يمكننا معرفة حجم قيمة string slice في وقت التجميع (compile time): إنه ضعف طول `usize`. أي أننا نعرف دائماً حجم string slice، بغض النظر عن طول السلسلة النصية التي يشير إليها. بشكل عام، هذه هي الطريقة التي تُستخدم بها dynamically sized types في Rust: لديها قدر إضافي من البيانات الوصفية (metadata) التي تخزن حجم المعلومات الديناميكية. القاعدة الذهبية للأنواع ذات الحجم الديناميكي هي أنه يجب علينا دائماً وضع قيم DSTs خلف مؤشر (pointer) من نوع ما.

يمكننا دمج `str` مع جميع أنواع الـ pointers: على سبيل المثال، `Box<str>` أو `Rc<str>`. في الواقع، لقد رأيت هذا من قبل ولكن مع نوع مختلف ذو حجم ديناميكي: السمات (traits). كل trait هو DST يمكننا الرجوع إليه باستخدام اسم الـ trait. في قسم ["استخدام كائنات السمات للتجريد فوق السلوك المشترك"][using-trait-objects-to-abstract-over-shared-behavior] في الفصل الثامن عشر، ذكرنا أنه لاستخدام traits ككائنات سمات (trait objects)، يجب وضعها خلف pointer، مثل `&dyn Trait` أو `Box<dyn Trait>` (و `Rc<dyn Trait>` سيعمل أيضاً).

للعمل مع DSTs، توفر Rust سمة الـ `Sized` لتحديد ما إذا كان حجم النوع معروفاً في compile time أم لا. يتم تطبيق هذه الـ trait تلقائياً لكل شيء يُعرف حجمه في compile time. بالإضافة إلى ذلك، تضيف Rust ضمناً قيداً (bound) على `Sized` لكل دالة عامة (generic function). أي أن تعريف generic function مثل هذا:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-12-generic-fn-definition/src/lib.rs}}
```

يتم التعامل معه فعلياً كما لو كنا قد كتبنا هذا:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-13-generic-implicit-sized-bound/src/lib.rs}}
```

بشكل افتراضي، ستعمل generic functions فقط على الأنواع التي لها حجم معروف في compile time. ومع ذلك، يمكنك استخدام الـ syntax الخاص التالي لتخفيف هذا القيد:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-14-generic-maybe-sized/src/lib.rs}}
```

قيد الـ trait على `?Sized` يعني "`T` قد يكون أو لا يكون `Sized` "، وهذا الترميز يتجاوز الافتراضي بأن الأنواع العامة يجب أن يكون لها حجم معروف في compile time. الـ syntax الخاص بـ `?Trait` بهذا المعنى متاح فقط لـ `Sized` ، وليس لأي traits أخرى.

لاحظ أيضاً أننا قمنا بتغيير نوع المعامل `t` من `T` إلى `&T`. ولأن النوع قد لا يكون `Sized` ، نحتاج إلى استخدامه خلف نوع من الـ pointers. في هذه الحالة، اخترنا مرجعاً (reference).

بعد ذلك، سنتحدث عن الدوال والإغلاقات (closures)!

[encapsulation-that-hides-implementation-details]: ch18-01-what-is-oo.html#encapsulation-that-hides-implementation-details
[string-slices]: ch04-03-slices.html#string-slices
[the-match-control-flow-construct]: ch06-02-match.html#the-match-control-flow-construct
[using-trait-objects-to-abstract-over-shared-behavior]: ch18-02-trait-objects.html#using-trait-objects-to-abstract-over-shared-behavior
[newtype]: ch20-02-advanced-traits.html#implementing-external-traits-with-the-newtype-pattern
```
