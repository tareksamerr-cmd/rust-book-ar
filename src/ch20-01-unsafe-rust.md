## لغة رست غير الآمنة (Unsafe Rust)

جميع الأكواد التي ناقشناها حتى الآن كانت تخضع لضمانات سلامة الذاكرة (memory safety guarantees) الخاصة بلغة رست (Rust) والتي يتم فرضها في وقت التصريف (compile time). ومع ذلك، تمتلك رست لغة ثانية مخفية بداخلها لا تفرض ضمانات سلامة الذاكرة هذه: تُسمى _رست غير الآمنة_ (unsafe Rust) وهي تعمل تماماً مثل رست العادية ولكنها تمنحنا "قوى خارقة" إضافية.

توجد Unsafe Rust لأن التحليل الساكن (static analysis)، بطبيعته، يكون متحفظاً. فعندما يحاول المصرّف (compiler) تحديد ما إذا كان الكود يلتزم بالضمانات أم لا، فمن الأفضل له رفض بعض البرامج الصالحة بدلاً من قبول بعض البرامج غير الصالحة. على الرغم من أن الكود _قد_ يكون سليماً، إلا أنه إذا لم يمتلك compiler معلومات كافية ليكون واثقاً، فإنه سيرفض الكود. في هذه الحالات، يمكنك استخدام كود غير آمن (unsafe code) لإخبار compiler: "ثق بي، أنا أعرف ما أفعله". ومع ذلك، كن حذراً، فأنت تستخدم Unsafe Rust على مسؤوليتك الخاصة: إذا استخدمت unsafe code بشكل غير صحيح، فقد تحدث مشكلات بسبب عدم سلامة الذاكرة، مثل فك مرجع مؤشر فارغ (null pointer dereferencing).

سبب آخر لامتلاك رست وجهاً آخر غير آمن هو أن عتاد الحاسوب (computer hardware) الأساسي غير آمن بطبيعته. إذا لم تسمح لك رست بإجراء عمليات غير آمنة (unsafe operations)، فلن تتمكن من أداء مهام معينة. تحتاج رست للسماح لك بالقيام ببرمجة الأنظمة منخفضة المستوى (low-level systems programming)، مثل التفاعل المباشر مع نظام التشغيل (operating system) أو حتى كتابة نظام التشغيل الخاص بك. العمل مع low-level systems programming هو أحد أهداف اللغة. دعونا نستكشف ما يمكننا فعله باستخدام Unsafe Rust وكيفية القيام بذلك.

<!-- Old headings. Do not remove or links may break. -->

<a id="unsafe-superpowers"></a>

### ممارسة القوى الخارقة غير الآمنة (Performing Unsafe Superpowers)

للتحول إلى Unsafe Rust، استخدم الكلمة المفتاحية (keyword) `unsafe` ثم ابدأ كتلة (block) جديدة تحتوي على unsafe code. يمكنك اتخاذ خمسة إجراءات في Unsafe Rust لا يمكنك القيام بها في رست الآمنة (safe Rust)، والتي نسميها _القوى الخارقة غير الآمنة_ (unsafe superpowers). تشمل هذه القوى القدرة على:

1. فك مرجع مؤشر خام (Dereference a raw pointer).
1. استدعاء دالة (function) أو دالة كائن (method) غير آمنة.
1. الوصول إلى متغير ساكن قابل للتغيير (mutable static variable) أو تعديله.
1. تنفيذ سمة (trait) غير آمنة.
1. الوصول إلى حقول الاتحادات (union).

من المهم فهم أن `unsafe` لا توقف عمل فاحص الاستعارة (borrow checker) أو تعطّل أي من فحوصات السلامة الأخرى في رست: إذا استخدمت مرجعاً (reference) في unsafe code، فسيظل خاضعاً للفحص. تمنحك keyword `unsafe` فقط إمكانية الوصول إلى هذه الميزات الخمس التي لا يتم فحصها بواسطة compiler من حيث سلامة الذاكرة. ستظل تحصل على درجة معينة من السلامة داخل unsafe block.

بالإضافة إلى ذلك، لا تعني `unsafe` أن الكود داخل block هو بالضرورة خطير أو أنه سيواجه بالتأكيد مشكلات في سلامة الذاكرة: القصد هو أنك كمبرمج ستضمن أن الكود داخل `unsafe` block سيصل إلى الذاكرة بطريقة صالحة.

البشر معرضون للخطأ وستحدث الأخطاء، ولكن من خلال اشتراط أن تكون هذه العمليات الخمس غير الآمنة داخل blocks مميزة بـ `unsafe` ، ستعرف أن أي أخطاء تتعلق بسلامة الذاكرة يجب أن تكون داخل `unsafe` block. اجعل `unsafe` blocks صغيرة؛ ستكون ممتناً لذلك لاحقاً عندما تحقق في أخطاء الذاكرة (memory bugs).

لعزل unsafe code قدر الإمكان، من الأفضل إحاطة هذا الكود داخل تجريد آمن (safe abstraction) وتوفير واجهة برمجة تطبيقات (API) آمنة، وهو ما سنناقشه لاحقاً في هذا الفصل عندما نفحص unsafe functions و methods. يتم تنفيذ أجزاء من المكتبة القياسية (standard library) كتجريدات آمنة فوق unsafe code تم تدقيقه. إن تغليف unsafe code في safe abstraction يمنع استخدامات `unsafe` من التسرب إلى جميع الأماكن التي قد ترغب أنت أو مستخدموك في استخدام الوظيفة المنفذة باستخدام unsafe code، لأن استخدام safe abstraction هو أمر آمن.

دعونا نلقي نظرة على كل من القوى الخارقة الخمس غير الآمنة بالترتيب. سننظر أيضاً في بعض التجريدات التي توفر واجهة آمنة لـ unsafe code.

### فك مرجع مؤشر خام (Dereferencing a Raw Pointer)

في الفصل الرابع، في قسم ["المراجع المعلقة" (Dangling References)][dangling-references]، ذكرنا أن compiler يضمن أن المراجع صالحة دائماً. تمتلك Unsafe Rust نوعين جديدين يسمى كل منهما _مؤشر خام_ (raw pointer) يشبهان المراجع. كما هو الحال مع المراجع، يمكن أن تكون raw pointers غير قابلة للتغيير (immutable) أو قابلة للتغيير (mutable) وتُكتب كـ `*const T` و `*mut T` على التوالي. النجمة ليست عامل فك المرجع (dereference operator)؛ بل هي جزء من اسم النوع. في سياق raw pointers، تعني _immutable_ أنه لا يمكن التعيين للمؤشر مباشرة بعد فك مرجعه.

على عكس المراجع والمؤشرات الذكية (smart pointers)، فإن raw pointers:

- يُسمح لها بتجاهل قواعد الاستعارة (borrowing rules) من خلال امتلاك مؤشرات immutable و mutable معاً أو عدة مؤشرات mutable لنفس الموقع.
- لا تضمن الإشارة إلى ذاكرة صالحة.
- يُسمح لها بأن تكون فارغة (null).
- لا تنفذ أي تنظيف تلقائي (automatic cleanup).

من خلال اختيار عدم فرض رست لهذه الضمانات، يمكنك التخلي عن السلامة المضمونة مقابل أداء أفضل أو القدرة على التفاعل مع لغة أخرى أو عتاد لا تنطبق عليه ضمانات رست.

توضح القائمة 20-1 كيفية إنشاء raw pointer غير قابل للتغيير وآخر قابل للتغيير.

<Listing number="20-1" caption="إنشاء مؤشرات خام باستخدام عوامل الاستعارة الخام">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-01/src/main.rs:here}}
```

</Listing>

لاحظ أننا لا ندرج keyword `unsafe` في هذا الكود. يمكننا إنشاء raw pointers في safe code؛ لكننا لا نستطيع فك مرجع raw pointers خارج unsafe block، كما سترى بعد قليل.

لقد أنشأنا raw pointers باستخدام عوامل الاستعارة الخام (raw borrow operators): `&raw const num` ينشئ مؤشر خام `*const i32` غير قابل للتغيير، و `&raw mut num` ينشئ مؤشر خام `*mut i32` قابل للتغيير. ولأننا أنشأناها مباشرة من متغير محلي، فنحن نعلم أن هذه الـ raw pointers المحددة صالحة، لكن لا يمكننا وضع هذا الافتراض حول أي raw pointer.

لإثبات ذلك، سنقوم بعد ذلك بإنشاء raw pointer لا يمكننا التأكد من صلاحيته، باستخدام keyword `as` لتحويل (cast) قيمة بدلاً من استخدام raw borrow operator. توضح القائمة 20-2 كيفية إنشاء raw pointer لموقع عشوائي في الذاكرة. محاولة استخدام ذاكرة عشوائية هو أمر غير محدد (undefined): قد تكون هناك بيانات في ذلك العنوان وقد لا تكون، وقد يقوم compiler بتحسين الكود بحيث لا يكون هناك وصول للذاكرة، أو قد ينتهي البرنامج بخطأ في التجزئة (segmentation fault). عادةً، لا يوجد سبب وجيه لكتابة كود كهذا، خاصة في الحالات التي يمكنك فيها استخدام raw borrow operator بدلاً من ذلك، ولكن الأمر ممكن.

<Listing number="20-2" caption="إنشاء مؤشر خام لعنوان ذاكرة عشوائي">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-02/src/main.rs:here}}
```

</Listing>

تذكر أنه يمكننا إنشاء raw pointers في safe code، لكن لا يمكننا فك مرجع raw pointers وقراءة البيانات التي تشير إليها. في القائمة 20-3، نستخدم dereference operator `*` على raw pointer مما يتطلب unsafe block.

<Listing number="20-3" caption="فك مرجع المؤشرات الخام داخل كتلة unsafe">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-03/src/main.rs:here}}
```

</Listing>

إنشاء مؤشر لا يسبب ضرراً؛ فقط عندما نحاول الوصول إلى القيمة التي يشير إليها قد ينتهي بنا الأمر بالتعامل مع قيمة غير صالحة.

لاحظ أيضاً أنه في القائمتين 20-1 و 20-3، أنشأنا raw pointers من نوع `*const i32` و `*mut i32` يشيران كلاهما إلى نفس موقع الذاكرة حيث يتم تخزين `num`. إذا حاولنا بدلاً من ذلك إنشاء reference غير قابل للتغيير وآخر قابل للتغيير لـ `num` ، فلن يتم تصريف الكود لأن قواعد الملكية (ownership rules) في رست لا تسمح بمرجع mutable في نفس وقت وجود أي مراجع immutable. باستخدام raw pointers، يمكننا إنشاء مؤشر mutable ومؤشر immutable لنفس الموقع وتغيير البيانات من خلال المؤشر mutable، مما قد يؤدي إلى حدوث سباق بيانات (data race). كن حذراً!

مع كل هذه المخاطر، لماذا قد تستخدم raw pointers؟ أحد حالات الاستخدام الرئيسية هو عند التفاعل مع كود لغة C، كما سترى في القسم التالي. حالة أخرى هي عند بناء safe abstractions لا يفهمها borrow checker. سنقدم unsafe functions ثم ننظر في مثال لـ safe abstraction يستخدم unsafe code.

### استدعاء دالة أو دالة كائن غير آمنة (Calling an Unsafe Function or Method)

النوع الثاني من العمليات التي يمكنك القيام بها في unsafe block هو استدعاء unsafe functions. تبدو unsafe functions و methods تماماً مثل functions و methods العادية، لكنها تحتوي على `unsafe` إضافية قبل بقية التعريف. تشير keyword `unsafe` في هذا السياق إلى أن function لها متطلبات نحتاج إلى الالتزام بها عند استدعائها، لأن رست لا تستطيع ضمان وفائنا بهذه المتطلبات. من خلال استدعاء unsafe function داخل `unsafe` block، فإننا نقول إننا قرأنا توثيق هذه function ونتحمل مسؤولية الالتزام بعقودها.

إليك unsafe function تسمى `dangerous` لا تفعل شيئاً في جسمها:

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-01-unsafe-fn/src/main.rs:here}}
```

يجب علينا استدعاء function `dangerous` داخل `unsafe` block منفصل. إذا حاولنا استدعاء `dangerous` بدون `unsafe` block، فسنحصل على خطأ:

```console
{{#include ../listings/ch20-advanced-features/output-only-01-missing-unsafe/output.txt}}
```

باستخدام `unsafe` block، نحن نؤكد لرست أننا قرأنا توثيق function، ونفهم كيفية استخدامها بشكل صحيح، وتحققنا من أننا نفي بعقد function.

للقيام بعمليات غير آمنة في جسم unsafe function، لا تزال بحاجة إلى استخدام `unsafe` block، تماماً كما هو الحال داخل function عادية، وسيقوم compiler بتنبيهك إذا نسيت. يساعدنا هذا في إبقاء `unsafe` blocks صغيرة قدر الإمكان، حيث قد لا تكون العمليات غير آمنة مطلوبة عبر جسم function بالكامل.

#### إنشاء تجريد آمن فوق كود غير آمن (Creating a Safe Abstraction over Unsafe Code)

مجرد احتواء function على unsafe code لا يعني أننا بحاجة إلى تمييز function بأكملها كغير آمنة. في الواقع، يعد تغليف unsafe code في safe function تجريداً شائعاً. كمثال، دعونا ندرس function `split_at_mut` من standard library، والتي تتطلب بعض unsafe code. سنستكشف كيف يمكننا تنفيذها. يتم تعريف هذه method الآمنة على الشرائح القابلة للتغيير (mutable slices): فهي تأخذ شريحة (slice) واحدة وتجعلها شريحتين عن طريق تقسيم slice عند الفهرس (index) المعطى كمعامل (argument). توضح القائمة 20-4 كيفية استخدام `split_at_mut`.

<Listing number="20-4" caption="استخدام دالة split_at_mut الآمنة">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-04/src/main.rs:here}}
```

</Listing>

لا يمكننا تنفيذ هذه function باستخدام safe Rust فقط. قد تبدو المحاولة شيئاً مثل القائمة 20-5، والتي لن يتم تصريفها. للتبسيط، سنقوم بتنفيذ `split_at_mut` كـ function بدلاً من method وفقط لـ slices من قيم `i32` بدلاً من النوع العام (generic type) `T`.
```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-05/src/main.rs:here}}
```

</Listing>

تقوم هذه function أولاً بالحصول على الطول الإجمالي لـ slice. بعد ذلك، تتحقق (assert) من أن index المعطى كمعامل يقع ضمن slice من خلال التأكد مما إذا كان أقل من أو يساوي الطول. يعني هذا التحقق أنه إذا مررنا index أكبر من الطول لتقسيم slice عنده، فإن function ستتوقف بشكل طارئ (panic) قبل أن تحاول استخدام ذلك index.

بعد ذلك، نعيد شريحتين قابلتين للتغيير في صف (tuple): واحدة من بداية slice الأصلية إلى index المسمى `mid` والأخرى من `mid` إلى نهاية slice.

عندما نحاول تصريف الكود في القائمة 20-5، سنحصل على خطأ:

```console
{{#include ../listings/ch20-advanced-features/listing-20-05/output.txt}}
```

لا يستطيع borrow checker في رست فهم أننا نستعير أجزاء مختلفة من slice؛ فهو يعرف فقط أننا نستعير من نفس slice مرتين. استعارة أجزاء مختلفة من slice هو أمر سليم جوهرياً لأن الشريحتين لا تتداخلان، لكن رست ليست ذكية بما يكفي لمعرفة ذلك. عندما نعلم أن الكود سليم، ولكن رست لا تعلم ذلك، يحين الوقت للاستعانة بـ unsafe code.

توضح القائمة 20-6 كيفية استخدام unsafe block، و raw pointer، وبعض الاستدعاءات لـ unsafe functions لجعل تنفيذ `split_at_mut` يعمل.

<Listing number="20-6" caption="استخدام كود غير آمن في تنفيذ دالة split_at_mut">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-06/src/main.rs:here}}
```

</Listing>

تذكر من قسم ["نوع الشريحة" (The Slice Type)][the-slice-type] في الفصل الرابع أن slice هي مؤشر لبعض البيانات وطول slice. نحن نستخدم method المسمى `len` للحصول على طول slice و method المسمى `as_mut_ptr` للوصول إلى raw pointer الخاص بـ slice. في هذه الحالة، ولأن لدينا mutable slice لقيم `i32` ، فإن `as_mut_ptr` تعيد raw pointer من نوع `*mut i32` ، والذي قمنا بتخزينه في المتغير `ptr`.

نحتفظ بالتحقق من أن index المسمى `mid` يقع ضمن slice. بعد ذلك، نصل إلى unsafe code: تأخذ function المسمى `slice::from_raw_parts_mut` مؤشراً خاماً وطولاً، وتنشئ slice. نستخدم هذه function لإنشاء slice تبدأ من `ptr` وطولها `mid` من العناصر. بعد ذلك، نستدعي method المسمى `add` على `ptr` مع تمرير `mid` كمعامل للحصول على raw pointer يبدأ عند `mid` ، وننشئ slice باستخدام ذلك المؤشر وعدد العناصر المتبقية بعد `mid` كطول.

تعتبر function المسمى `slice::from_raw_parts_mut` غير آمنة لأنها تأخذ raw pointer ويجب أن تثق في أن هذا المؤشر صالح. كما أن method المسمى `add` على raw pointers هو أيضاً غير آمن لأنه يجب أن يثق في أن موقع الإزاحة (offset) هو أيضاً مؤشر صالح. لذلك، كان علينا وضع unsafe block حول استدعاءاتنا لـ `slice::from_raw_parts_mut` و `add` حتى نتمكن من استدعائهما. من خلال النظر في الكود وإضافة التحقق من أن `mid` يجب أن يكون أقل من أو يساوي `len` ، يمكننا القول إن جميع raw pointers المستخدمة داخل unsafe block ستكون مؤشرات صالحة لبيانات داخل slice. هذا استخدام مقبول ومناسب لـ `unsafe`.

لاحظ أننا لسنا بحاجة لتمييز function الناتجة `split_at_mut` كـ `unsafe` ، ويمكننا استدعاء هذه function من safe Rust. لقد أنشأنا safe abstraction لكود غير آمن مع تنفيذ لـ function يستخدم unsafe code بطريقة آمنة، لأنه ينشئ فقط مؤشرات صالحة من البيانات التي تملك هذه function صلاحية الوصول إليها.

في المقابل، فإن استخدام `slice::from_raw_parts_mut` في القائمة 20-7 سيؤدي على الأرجح إلى انهيار البرنامج عند استخدام slice. يأخذ هذا الكود موقعاً عشوائياً في الذاكرة وينشئ slice بطول 10,000 عنصر.

<Listing number="20-7" caption="إنشاء شريحة من موقع ذاكرة عشوائي">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-07/src/main.rs:here}}
```

</Listing>

نحن لا نملك الذاكرة في هذا الموقع العشوائي، ولا يوجد ضمان بأن slice التي ينشئها هذا الكود تحتوي على قيم `i32` صالحة. محاولة استخدام `values` كما لو كانت slice صالحة تؤدي إلى سلوك غير محدد (undefined behavior).

#### استخدام دوال خارجية (extern) لاستدعاء كود خارجي (Using extern Functions to Call External Code)

أحياناً قد يحتاج كود رست الخاص بك إلى التفاعل مع كود مكتوب بلغة أخرى. لهذا الغرض، تمتلك رست keyword `extern` التي تسهل إنشاء واستخدام _واجهة الدوال الأجنبية_ (Foreign Function Interface - FFI)، وهي طريقة للغة برمجة لتعريف functions وتمكين لغة برمجة مختلفة (أجنبية) من استدعاء تلك functions.

توضح القائمة 20-8 كيفية إعداد تكامل مع function المسمى `abs` من مكتبة C القياسية. الدوال المعلن عنها داخل blocks من نوع `extern` هي بشكل عام غير آمنة للاستدعاء من كود رست، لذا يجب أيضاً تمييز `extern` blocks بـ `unsafe`. والسبب هو أن اللغات الأخرى لا تفرض قواعد وضمانات رست، ولا تستطيع رست التحقق منها، لذا تقع المسؤولية على عاتق المبرمج لضمان السلامة.

<Listing number="20-8" file-name="src/main.rs" caption="الإعلان عن دالة خارجية (extern) معرفة بلغة أخرى واستدعاؤها">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-08/src/main.rs}}
```

</Listing>

داخل block من نوع `unsafe extern "C"` ، ندرج أسماء وتواقيع (signatures) الدوال الخارجية من لغة أخرى نريد استدعاءها. يحدد جزء `"C"` أي _واجهة ثنائية للتطبيق_ (Application Binary Interface - ABI) تستخدمها function الخارجية: تحدد ABI كيفية استدعاء function على مستوى لغة التجميع (assembly). تعتبر ABI الخاصة بـ `"C"` هي الأكثر شيوعاً وتتبع ABI الخاصة بلغة البرمجة C. تتوفر معلومات حول جميع واجهات ABI التي تدعمها رست في [مرجع رست (Rust Reference)][ABI].

كل عنصر يتم الإعلان عنه داخل `unsafe extern` block هو غير آمن ضمنياً. ومع ذلك، فإن بعض functions الخاصة بـ FFI *آمنة* للاستدعاء. على سبيل المثال، function المسمى `abs` من مكتبة C القياسية ليس لديها أي اعتبارات تتعلق بسلامة الذاكرة، ونحن نعلم أنه يمكن استدعاؤها مع أي `i32`. في حالات كهذه، يمكننا استخدام keyword `safe` للقول إن هذه function المحددة آمنة للاستدعاء على الرغم من وجودها في `unsafe extern` block. بمجرد إجراء هذا التغيير، لن يتطلب استدعاؤها وجود unsafe block، كما هو موضح في القائمة 20-9.

<Listing number="20-9" file-name="src/main.rs" caption="تمييز دالة صراحة كـ safe داخل كتلة unsafe extern واستدعاؤها بأمان">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-09/src/main.rs}}
```

</Listing>

تمييز function كـ `safe` لا يجعلها آمنة بطبيعتها! بدلاً من ذلك، هو بمثابة وعد تقطعه لرست بأنها آمنة. تظل مسؤوليتك هي التأكد من الوفاء بهذا الوعد!

#### استدعاء دوال رست من لغات أخرى (Calling Rust Functions from Other Languages)

يمكننا أيضاً استخدام `extern` لإنشاء واجهة تسمح للغات أخرى باستدعاء functions الخاصة برست. بدلاً من إنشاء `extern` block كامل، نضيف keyword `extern` ونحدد ABI المراد استخدامها قبل keyword `fn` لـ function المعنية مباشرة. نحتاج أيضاً إلى إضافة تعليق توضيحي (annotation) من نوع `#[unsafe(no_mangle)]` لإخبار compiler رست بعدم تشويه (mangle) اسم هذه function. _التشويه_ (Mangling) هو عندما يقوم compiler بتغيير الاسم الذي أعطيناه لـ function إلى اسم مختلف يحتوي على مزيد من المعلومات لتستهلكها أجزاء أخرى من عملية التصريف ولكنه أقل قابلية للقراءة من قبل البشر. يقوم كل compiler لغة برمجة بتشويه الأسماء بشكل مختلف قليلاً، لذا لكي تكون function رست قابلة للتسمية من قبل لغات أخرى، يجب علينا تعطيل تشويه الأسماء الخاص بـ compiler رست. هذا أمر غير آمن لأنه قد تحدث تصادمات في الأسماء عبر المكتبات بدون mangling المدمج، لذا تقع على عاتقنا مسؤولية التأكد من أن الاسم الذي نختاره آمن للتصدير بدون mangling.

في المثال التالي، نجعل function المسمى `call_from_c` قابلة للوصول من كود C، بعد تصريفها إلى مكتبة مشتركة (shared library) وربطها من C:

```
#[unsafe(no_mangle)]
pub extern "C" fn call_from_c() {
    println!("Just called a Rust function from C!");
}
```

هذا الاستخدام لـ `extern` يتطلب `unsafe` فقط في السمة (attribute)، وليس على `extern` block.

### الوصول إلى متغير ساكن قابل للتغيير أو تعديله (Accessing or Modifying a Mutable Static Variable)

في هذا الكتاب، لم نتحدث بعد عن المتغيرات العامة (global variables)، والتي تدعمها رست ولكنها قد تكون إشكالية مع قواعد الملكية في رست. إذا كان هناك خيطان (threads) يصلان إلى نفس المتغير العام القابل للتغيير، فقد يتسبب ذلك في data race.

في رست، تسمى global variables بالمتغيرات _الساكنة_ (static variables). توضح القائمة 20-10 مثالاً للإعلان عن static variable واستخدامه مع شريحة نصية (string slice) كقيمة.

<Listing number="20-10" file-name="src/main.rs" caption="تعريف متغير ساكن غير قابل للتغيير واستخدامه">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-10/src/main.rs}}
```

</Listing>

تشبه static variables الثوابت (constants)، التي ناقشناها في قسم ["الإعلان عن الثوابت" (Declaring Constants)][constants] في الفصل الثالث. تكون أسماء static variables بصيغة `SCREAMING_SNAKE_CASE` حسب العرف. يمكن لـ static variables فقط تخزين مراجع ذات عمر `'static` (static lifetime)، مما يعني أن compiler رست يمكنه معرفة العمر (lifetime) ولسنا مطالبين بتمييزه صراحة. الوصول إلى static variable غير قابل للتغيير هو أمر آمن.

هناك فرق دقيق بين constants و static variables غير القابلة للتغيير وهو أن القيم في static variable لها عنوان ثابت في الذاكرة. استخدام القيمة سيصل دائماً إلى نفس البيانات. من ناحية أخرى، يُسمح لـ constants بتكرار بياناتها كلما تم استخدامها. فرق آخر هو أن static variables يمكن أن تكون قابلة للتغيير (mutable). الوصول إلى static variables القابلة للتغيير وتعديلها هو أمر _غير آمن_ (unsafe). توضح القائمة 20-11 كيفية الإعلان عن static variable قابل للتغيير يسمى `COUNTER` والوصول إليه وتعديله.

<Listing number="20-11" file-name="src/main.rs" caption="القراءة من متغير ساكن قابل للتغيير أو الكتابة فيه هو أمر غير آمن.">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-11/src/main.rs}}
```

</Listing>

كما هو الحال مع المتغيرات العادية، نحدد القابلية للتغيير باستخدام keyword `mut`. أي كود يقرأ من `COUNTER` أو يكتب فيه يجب أن يكون داخل unsafe block. الكود في القائمة 20-11 يتم تصريفه ويطبع `COUNTER: 3` كما نتوقع لأنه يعمل بخيط واحد (single threaded). من المحتمل أن يؤدي وصول عدة threads إلى `COUNTER` إلى حدوث data races، لذا فهو undefined behavior. لذلك، نحتاج إلى تمييز function بأكملها كـ `unsafe` وتوثيق قيود السلامة بحيث يعرف أي شخص يستدعي function ما هو مسموح له وما هو غير مسموح له بفعله بأمان.

كلما كتبنا unsafe function، فمن المعتاد (idiomatic) كتابة تعليق يبدأ بـ `SAFETY` ويشرح ما يحتاج المستدعي فعله لاستدعاء function بأمان. وبالمثل، كلما قمنا بعملية غير آمنة، فمن المعتاد كتابة تعليق يبدأ بـ `SAFETY` لشرح كيفية الالتزام بقواعد السلامة.

بالإضافة إلى ذلك، سيرفض compiler افتراضياً أي محاولة لإنشاء مراجع لـ static variable قابل للتغيير من خلال قاعدة تحقق (lint) في compiler. يجب عليك إما إلغاء الاشتراك صراحة في حماية lint هذه عن طريق إضافة annotation من نوع `#[allow(static_mut_refs)]` أو الوصول إلى static variable القابل للتغيير عبر raw pointer تم إنشاؤه باستخدام أحد raw borrow operators. يتضمن ذلك الحالات التي يتم فيها إنشاء reference بشكل غير مرئي، كما هو الحال عند استخدامه في `println!` في هذه القائمة. يساعد اشتراط إنشاء مراجع لـ static mutable variables عبر raw pointers في جعل متطلبات السلامة لاستخدامها أكثر وضوحاً.

مع البيانات القابلة للتغيير التي يمكن الوصول إليها عالمياً، من الصعب ضمان عدم وجود data races، ولهذا السبب تعتبر رست أن static variables القابلة للتغيير غير آمنة. حيثما أمكن، يفضل استخدام تقنيات التزامن (concurrency) والمؤشرات الذكية الآمنة للخيوط (thread-safe smart pointers) التي ناقشناها في الفصل 16 بحيث يتحقق compiler من أن الوصول إلى البيانات من threads مختلفة يتم بأمان.
### تنفيذ سمة غير آمنة (Implementing an Unsafe Trait)

يمكننا استخدام `unsafe` لتنفيذ سمة غير آمنة (unsafe trait). تكون trait غير آمنة عندما يكون لواحد على الأقل من methods الخاصة بها بعض الثوابت (invariants) التي لا يستطيع compiler التحقق منها. نعلن أن trait هي `unsafe` عن طريق إضافة keyword `unsafe` قبل `trait` وتمييز تنفيذ trait كـ `unsafe` أيضاً، كما هو موضح في القائمة 20-12.

<Listing number="20-12" caption="تعريف سمة غير آمنة وتنفيذها">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-12/src/main.rs:here}}
```

</Listing>

باستخدام `unsafe impl` ، نحن نعد بأننا سنلتزم بـ invariants التي لا يستطيع compiler التحقق منها.

كمثال، تذكر سمات العلامات (marker traits) المسمى `Send` و `Sync` التي ناقشناها في قسم ["التزامن القابل للتوسع مع Send و Sync"][send-and-sync] في الفصل 16: يقوم compiler بتنفيذ هذه traits تلقائياً إذا كانت أنواعنا تتكون بالكامل من أنواع أخرى تنفذ `Send` و `Sync`. إذا قمنا بتنفيذ نوع يحتوي على نوع لا ينفذ `Send` أو `Sync` ، مثل raw pointers، وأردنا تمييز هذا النوع كـ `Send` أو `Sync` ، فيجب علينا استخدام `unsafe`. لا تستطيع رست التحقق من أن نوعنا يلتزم بالضمانات التي تسمح بإرساله بأمان عبر threads أو الوصول إليه من عدة threads؛ لذلك، نحتاج إلى إجراء تلك الفحوصات يدوياً والإشارة إلى ذلك باستخدام `unsafe`.

### الوصول إلى حقول الاتحاد (Accessing Fields of a Union)

الإجراء الأخير الذي يعمل فقط مع `unsafe` هو الوصول إلى حقول الاتحاد (union). يشبه *union* الهيكل (struct)، ولكن يتم استخدام حقل واحد فقط معلن عنه في مثيل (instance) معين في وقت واحد. تُستخدم unions بشكل أساسي للتفاعل مع unions في كود لغة C. الوصول إلى حقول union غير آمن لأن رست لا تستطيع ضمان نوع البيانات المخزنة حالياً في instance الخاص بـ union. يمكنك معرفة المزيد عن unions في [مرجع رست (Rust Reference)][unions].

### استخدام Miri للتحقق من الكود غير الآمن (Using Miri to Check Unsafe Code)

عند كتابة unsafe code، قد ترغب في التحقق من أن ما كتبته هو بالفعل آمن وصحيح. أحد أفضل الطرق للقيام بذلك هو استخدام Miri، وهي أداة رسمية من رست لاكتشاف undefined behavior. بينما يعتبر borrow checker أداة _ساكنة_ (static tool) تعمل في وقت التصريف، فإن Miri هي أداة _ديناميكية_ (dynamic tool) تعمل في وقت التشغيل (runtime). تقوم بفحص الكود الخاص بك عن طريق تشغيل برنامجك، أو مجموعة الاختبارات الخاصة به، واكتشاف متى تنتهك القواعد التي تفهمها حول كيفية عمل رست.

يتطلب استخدام Miri نسخة ليلية (nightly build) من رست (والتي نتحدث عنها أكثر في [الملحق G: كيف تُصنع رست و "رست الليلية"][nightly]). يمكنك تثبيت كل من نسخة nightly من رست وأداة Miri عن طريق كتابة `rustup +nightly component add miri`. هذا لا يغير إصدار رست الذي يستخدمه مشروعك؛ بل يضيف الأداة فقط إلى نظامك حتى تتمكن من استخدامها عندما تريد. يمكنك تشغيل Miri على مشروع عن طريق كتابة `cargo +nightly miri run` أو `cargo +nightly miri test`.

كمثال على مدى فائدة ذلك، انظر ماذا يحدث عندما نقوم بتشغيلها ضد القائمة 20-7.

```console
{{#include ../listings/ch20-advanced-features/listing-20-07/output.txt}}
```

تحذرنا Miri بشكل صحيح من أننا نقوم بتحويل (casting) عدد صحيح إلى مؤشر، وهو ما قد يمثل مشكلة، لكن Miri لا تستطيع تحديد ما إذا كانت هناك مشكلة لأنها لا تعرف مصدر المؤشر. بعد ذلك، تعيد Miri خطأً حيث يوجد في القائمة 20-7 سلوك غير محدد لأن لدينا مؤشر معلق (dangling pointer). بفضل Miri، نعلم الآن أن هناك خطراً من حدوث undefined behavior، ويمكننا التفكير في كيفية جعل الكود آمناً. في بعض الحالات، يمكن لـ Miri تقديم توصيات حول كيفية إصلاح الأخطاء.

لا تلتقط Miri كل ما قد تخطئ فيه عند كتابة unsafe code. Miri هي أداة تحليل ديناميكي، لذا فهي تلتقط فقط المشكلات في الكود الذي يتم تشغيله بالفعل. هذا يعني أنك ستحتاج إلى استخدامها جنباً إلى جنب مع تقنيات اختبار جيدة لزيادة ثقتك في unsafe code الذي كتبته. كما أن Miri لا تغطي كل طريقة ممكنة يمكن أن يكون بها كودك غير سليم (unsound).

بمعنى آخر: إذا التقطت Miri مشكلة، فأنت تعلم أن هناك خطأ (bug)، ولكن مجرد عدم التقاط Miri لخطأ لا يعني عدم وجود مشكلة. ومع ذلك، يمكنها التقاط الكثير. جرب تشغيلها على الأمثلة الأخرى لـ unsafe code في هذا الفصل وانظر ماذا ستقول!

يمكنك معرفة المزيد عن Miri في [مستودع GitHub الخاص بها][miri].

<!-- Old headings. Do not remove or links may break. -->

<a id="when-to-use-unsafe-code"></a>

### استخدام الكود غير الآمن بشكل صحيح (Using Unsafe Code Correctly)

استخدام `unsafe` لممارسة إحدى القوى الخارقة الخمس التي ناقشناها للتو ليس خطأً أو حتى أمراً غير مرغوب فيه، ولكن من الأصعب كتابة unsafe code بشكل صحيح لأن compiler لا يستطيع المساعدة في الحفاظ على سلامة الذاكرة. عندما يكون لديك سبب لاستخدام unsafe code، يمكنك القيام بذلك، ووجود التمييز الصريح بـ `unsafe` يجعل من السهل تتبع مصدر المشكلات عند حدوثها. كلما كتبت unsafe code، يمكنك استخدام Miri لمساعدتك على أن تكون أكثر ثقة في أن الكود الذي كتبته يلتزم بقواعد رست.

لاستكشاف أعمق بكثير حول كيفية العمل بفعالية مع Unsafe Rust، اقرأ دليل رست الرسمي لـ `unsafe` ، [The Rustonomicon][nomicon].

[dangling-references]: ch04-02-references-and-borrowing.html#dangling-references
[ABI]: ../reference/items/external-blocks.html#abi
[constants]: ch03-01-variables-and-mutability.html#declaring-constants
[send-and-sync]: ch16-04-extensible-concurrency-sync-and-send.html
[the-slice-type]: ch04-03-slices.html#the-slice-type
[unions]: ../reference/items/unions.html
[miri]: https://github.com/rust-lang/miri
[editions]: appendix-05-editions.html
[nightly]: appendix-07-nightly-rust.html
[nomicon]: https://doc.rust-lang.org/nomicon/
