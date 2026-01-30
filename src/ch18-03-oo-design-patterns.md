## تنفيذ نمط تصميم كائني التوجه (Implementing an Object-Oriented Design Pattern)

يُعد _نمط الحالة_ (state pattern) أحد أنماط التصميم كائنية التوجه (object-oriented design patterns). جوهر هذا النمط هو أننا نعرف مجموعة من الحالات التي يمكن أن تتخذها قيمة ما داخلياً. يتم تمثيل هذه الحالات بواسطة مجموعة من _كائنات الحالة_ (state objects)، ويتغير سلوك القيمة بناءً على حالتها. سنعمل من خلال مثال على هيكل (struct) لمنشور مدونة يحتوي على حقل للاحتفاظ بحالته، والتي ستكون state object من المجموعة "مسودة" (draft)، أو "مراجعة" (review)، أو "منشور" (published).

تتشارك state objects في الوظائف: في لغة رست (Rust)، بالطبع، نستخدم structs والسمات (traits) بدلاً من الكائنات (objects) والوراثة (inheritance). كل state object مسؤول عن سلوكه الخاص وعن تحديد متى يجب أن ينتقل إلى حالة أخرى. القيمة التي تحمل state object لا تعرف شيئاً عن السلوك المختلف للحالات أو متى يتم الانتقال بين الحالات.

ميزة استخدام state pattern هي أنه عندما تتغير متطلبات العمل (business requirements) للبرنامج، فلن نحتاج إلى تغيير كود القيمة التي تحمل الحالة أو الكود الذي يستخدم تلك القيمة. سنحتاج فقط إلى تحديث الكود داخل أحد state objects لتغيير قواعده أو ربما إضافة المزيد من state objects.

أولاً، سنقوم بتنفيذ state pattern بطريقة كائنية التوجه (object-oriented) تقليدية. بعد ذلك، سنستخدم نهجاً أكثر طبيعية في Rust. دعونا نبدأ في تنفيذ سير عمل (workflow) لمنشور مدونة تدريجياً باستخدام state pattern.

ستبدو الوظيفة النهائية كما يلي:

1. يبدأ منشور المدونة كمسودة فارغة.
1. عند الانتهاء من المسودة، يتم طلب مراجعة للمنشور.
1. عند الموافقة على المنشور، يتم نشره.
1. تعيد منشورات المدونة المنشورة فقط محتوى لطباعته بحيث لا يمكن نشر المنشورات غير المعتمدة عن طريق الخطأ.

أي تغييرات أخرى يتم محاولتها على المنشور يجب ألا يكون لها أي تأثير. على سبيل المثال، إذا حاولنا الموافقة على منشور مدونة في حالة draft قبل طلب review، فيجب أن يظل المنشور مسودة غير منشورة.

<!-- Old headings. Do not remove or links may break. -->

<a id="a-traditional-object-oriented-attempt"></a>

### محاولة النمط كائني التوجه التقليدي (Attempting Traditional Object-Oriented Style)

هناك طرق لا حصر لها لهيكلة الكود لحل نفس المشكلة، ولكل منها مقايضات (trade-offs) مختلفة. تنفيذ هذا القسم يتبع أسلوباً كائني التوجه تقليدياً، وهو أمر ممكن كتابته في Rust، ولكنه لا يستفيد من بعض نقاط قوة Rust. لاحقاً، سنعرض حلاً مختلفاً لا يزال يستخدم object-oriented design pattern ولكنه مهيكل بطريقة قد تبدو أقل مألوفة للمبرمجين ذوي الخبرة في object-oriented. سنقارن بين الحلين لتجربة trade-offs لتصميم كود Rust بشكل مختلف عن الكود في اللغات الأخرى.

توضح القائمة 18-11 سير العمل هذا في شكل كود: هذا مثال لاستخدام واجهة برمجة التطبيقات (API) التي سننفذها في حزمة مكتبة (library crate) تسمى `blog`. لن يتم تصريف هذا الكود بعد لأننا لم نقم بتنفيذ crate المسمى `blog`.

<Listing number="18-11" file-name="src/main.rs" caption="كود يوضح السلوك المطلوب الذي نريده لحزمة blog الخاصة بنا">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-11/src/main.rs:all}}
```

</Listing>

نريد السماح للمستخدم بإنشاء منشور مدونة جديد كمسودة باستخدام `Post::new`. ونريد السماح بإضافة نص إلى منشور المدونة. إذا حاولنا الحصول على محتوى المنشور فوراً، قبل الموافقة، فلا يجب أن نحصل على أي نص لأن المنشور لا يزال draft. لقد أضفنا `assert_eq!` في الكود لأغراض التوضيح. سيكون اختبار الوحدة (unit test) الممتاز لهذا هو التأكد من أن منشور المدونة في حالة draft يعيد سلسلة نصية فارغة من دالة الكائن (method) المسمى `content` ، لكننا لن نكتب اختبارات لهذا المثال.

بعد ذلك، نريد تمكين طلب مراجعة للمنشور، ونريد من `content` أن تعيد سلسلة نصية فارغة أثناء انتظار review. عندما يتلقى المنشور الموافقة، يجب أن يتم نشره، مما يعني أن نص المنشور سيتم إعادته عند استدعاء `content`.

لاحظ أن النوع الوحيد الذي نتفاعل معه من crate هو النوع `Post`. سيستخدم هذا النوع state pattern وسيحمل قيمة ستكون واحدة من ثلاثة state objects تمثل الحالات المختلفة التي يمكن أن يكون عليها المنشور—draft، أو review، أو published. سيتم إدارة التغيير من حالة إلى أخرى داخلياً ضمن النوع `Post`. تتغير الحالات استجابة لـ methods التي يستدعيها مستخدمو مكتبتنا على مثيل (instance) من `Post` ، لكن ليس عليهم إدارة تغييرات الحالة مباشرة. أيضاً، لا يمكن للمستخدمين ارتكاب خطأ في الحالات، مثل نشر منشور قبل مراجعته.

<!-- Old headings. Do not remove or links may break. -->

<a id="defining-post-and-creating-a-new-instance-in-the-draft-state"></a>

#### تعريف `Post` وإنشاء مثيل جديد (Defining Post and Creating a New Instance)

دعونا نبدأ في تنفيذ المكتبة! نحن نعلم أننا بحاجة إلى struct عام يسمى `Post` يحمل بعض المحتوى، لذا سنبدأ بتعريف struct ودالة عامة مرتبطة (associated function) تسمى `new` لإنشاء instance من `Post` ، كما هو موضح في القائمة 18-12. سنقوم أيضاً بإنشاء trait خاص يسمى `State` يحدد السلوك الذي يجب أن تمتلكه جميع state objects لـ `Post`.

بعد ذلك، سيحمل `Post` كائن سمة (trait object) من نوع `Box<dyn State>` داخل `Option<T>` في حقل خاص يسمى `state` للاحتفاظ بـ state object. سترى سبب ضرورة `Option<T>` بعد قليل.

<Listing number="18-12" file-name="src/lib.rs" caption="تعريف هيكل Post ودالة new التي تنشئ مثيلاً جديداً من Post، وسمة State، وهيكل Draft">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-12/src/lib.rs}}
```

</Listing>

تحدد trait المسمى `State` السلوك المشترك بين حالات المنشور المختلفة. كائنات الحالة هي `Draft` و `PendingReview` و `Published` ، وجميعها ستنفذ `State` trait. في الوقت الحالي، لا تحتوي trait على أي methods، وسنبدأ بتعريف حالة `Draft` فقط لأن هذه هي الحالة التي نريد أن يبدأ بها المنشور.

عندما ننشئ `Post` جديداً، نضبط حقل `state` الخاص به على قيمة `Some` تحمل `Box`. يشير هذا `Box` إلى instance جديد من struct المسمى `Draft`. يضمن هذا أنه كلما أنشأنا instance جديداً من `Post` ، فإنه سيبدأ كمسودة. نظرًا لأن حقل `state` في `Post` خاص، فلا توجد طريقة لإنشاء `Post` في أي حالة أخرى! في دالة `Post::new` ، نضبط حقل `content` على `String` جديد وفارغ.

#### تخزين نص محتوى المنشور (Storing the Text of the Post Content)

رأينا في القائمة 18-11 أننا نريد أن نكون قادرين على استدعاء method يسمى `add_text` وتمرير `&str` إليه ليتم إضافته كمحتوى نصي لمنشور المدونة. نقوم بتنفيذ هذا كـ method، بدلاً من كشف حقل `content` كـ `pub` ، حتى نتمكن لاحقاً من تنفيذ method يتحكم في كيفية قراءة بيانات حقل `content`. دالة `add_text` بسيطة للغاية، لذا دعونا نضيف التنفيذ في القائمة 18-13 إلى كتلة `impl Post`.

<Listing number="18-13" file-name="src/lib.rs" caption="تنفيذ دالة add_text لإضافة نص إلى محتوى المنشور">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-13/src/lib.rs:here}}
```

</Listing>

تأخذ `add_text` مرجعاً قابلاً للتغيير (mutable reference) لـ `self` لأننا نغير instance الخاص بـ `Post` الذي نستدعي `add_text` عليه. ثم نستدعي `push_str` على `String` في `content` ونمرر معامل (argument) النص لإضافته إلى `content` المحفوظ. هذا السلوك لا يعتمد على الحالة التي يمر بها المنشور، لذا فهو ليس جزءاً من state pattern. لا تتفاعل دالة `add_text` مع حقل `state` على الإطلاق، ولكنها جزء من السلوك الذي نريد دعمه.

<!-- Old headings. Do not remove or links may break. -->

<a id="ensuring-the-content-of-a-draft-post-is-empty"></a>

#### التأكد من أن محتوى المنشور المسودة فارغ (Ensuring That the Content of a Draft Post Is Empty)

حتى بعد استدعاء `add_text` وإضافة بعض المحتوى إلى منشورنا، لا نزال نريد أن تعيد دالة `content` شريحة نصية (string slice) فارغة لأن المنشور لا يزال في حالة draft، كما هو موضح في أول `assert_eq!` في القائمة 18-11. في الوقت الحالي، دعونا ننفذ دالة `content` بأبسط شيء يحقق هذا المتطلب: إعادة string slice فارغة دائماً. سنغير هذا لاحقاً بمجرد تنفيذ القدرة على تغيير حالة المنشور بحيث يمكن نشره. حتى الآن، يمكن أن تكون المنشورات في حالة draft فقط، لذا يجب أن يكون محتوى المنشور فارغاً دائماً. توضح القائمة 18-14 هذا التنفيذ المؤقت (placeholder).

<Listing number="18-14" file-name="src/lib.rs" caption="إضافة تنفيذ مؤقت لدالة content على Post تعيد دائماً شريحة نصية فارغة">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-14/src/lib.rs:here}}
```

</Listing>

مع إضافة دالة `content` هذه، يعمل كل شيء في القائمة 18-11 حتى أول `assert_eq!` كما هو مطلوب.

<!-- Old headings. Do not remove or links may break. -->

<a id="requesting-a-review-of-the-post-changes-its-state"></a>
<a id="requesting-a-review-changes-the-posts-state"></a>

#### طلب مراجعة، مما يغير حالة المنشور (Requesting a Review, Which Changes the Post’s State)

بعد ذلك، نحتاج إلى إضافة وظيفة لطلب مراجعة للمنشور، والتي يجب أن تغير حالته من `Draft` إلى `PendingReview`. توضح القائمة 18-15 هذا الكود.

<Listing number="18-15" file-name="src/lib.rs" caption="تنفيذ دوال request_review على Post وسمة State">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-15/src/lib.rs:here}}
```

</Listing>

نعطي `Post` دالة عامة تسمى `request_review` تأخذ mutable reference لـ `self`. ثم نستدعي دالة `request_review` داخلية على الحالة الحالية لـ `Post` ، وهذه الدالة الثانية تستهلك الحالة الحالية وتعيد حالة جديدة.

نضيف دالة `request_review` إلى `State` trait؛ ستحتاج جميع الأنواع التي تنفذ trait الآن إلى تنفيذ دالة `request_review`. لاحظ أنه بدلاً من وجود `self` أو `&self` أو `&mut self` كأول معامل للدالة، لدينا `self: Box<Self>`. تعني هذه الصيغة أن الدالة صالحة فقط عند استدعائها على `Box` يحمل النوع. تأخذ هذه الصيغة ملكية (ownership) لـ `Box<Self>` ، مما يؤدي إلى إبطال الحالة القديمة بحيث يمكن لقيمة الحالة في `Post` أن تتحول إلى حالة جديدة.

لاستهلاك الحالة القديمة، تحتاج دالة `request_review` إلى أخذ ملكية قيمة الحالة. وهنا يأتي دور `Option` في حقل `state` الخاص بـ `Post`: نستدعي دالة `take` لإخراج قيمة `Some` من حقل `state` وترك `None` في مكانها لأن Rust لا تسمح لنا بامتلاك حقول غير ممتلئة في structs. يتيح لنا ذلك نقل قيمة `state` خارج `Post` بدلاً من استعارتها. بعد ذلك، سنضبط قيمة `state` للمنشور على نتيجة هذه العملية.

نحتاج إلى ضبط `state` على `None` مؤقتاً بدلاً من ضبطها مباشرة
بعد أن قمنا بتحويلها إلى حالة جديدة.

تعيد دالة `request_review` في `Draft` مثيلاً جديداً ومغلفاً (boxed instance) من struct جديد يسمى `PendingReview` ، والذي يمثل الحالة عندما ينتظر المنشور المراجعة. ينفذ struct المسمى `PendingReview` أيضاً دالة `request_review` ولكنه لا يقوم بأي تحويلات. بدلاً من ذلك، يعيد نفسه لأنه عندما نطلب مراجعة لمنشور موجود بالفعل في حالة `PendingReview` ، يجب أن يظل في حالة `PendingReview`.

الآن يمكننا البدء في رؤية مزايا state pattern: دالة `request_review` في `Post` هي نفسها بغض النظر عن قيمة `state` الخاصة بها. كل حالة مسؤولة عن قواعدها الخاصة.

سنترك دالة `content` في `Post` كما هي، تعيد string slice فارغة. يمكننا الآن الحصول على `Post` في حالة `PendingReview` وكذلك في حالة `Draft` ، لكننا نريد نفس السلوك في حالة `PendingReview`. القائمة 18-11 تعمل الآن حتى استدعاء `assert_eq!` الثاني!

<!-- Old headings. Do not remove or links may break. -->

<a id="adding-the-approve-method-that-changes-the-behavior-of-content"></a>
<a id="adding-approve-to-change-the-behavior-of-content"></a>

#### إضافة `approve` لتغيير سلوك `content` (Adding approve to Change content's Behavior)

ستكون دالة `approve` مشابهة لدالة `request_review`: ستقوم بضبط `state` على القيمة التي تقول الحالة الحالية إنها يجب أن تمتلكها عند الموافقة على تلك الحالة، كما هو موضح في القائمة 18-16.

<Listing number="18-16" file-name="src/lib.rs" caption="تنفيذ دالة approve على Post وسمة State">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-16/src/lib.rs:here}}
```

</Listing>

نضيف دالة `approve` إلى `State` trait ونضيف struct جديداً ينفذ `State` ، وهو حالة `Published`.

بشكل مشابه للطريقة التي تعمل بها `request_review` في `PendingReview` ، إذا استدعينا دالة `approve` على `Draft` ، فلن يكون لها أي تأثير لأن `approve` ستعيد `self`. عندما نستدعي `approve` على `PendingReview` ، فإنها تعيد boxed instance جديداً من struct المسمى `Published`. ينفذ struct المسمى `Published` السمة `State` ، وبالنسبة لكل من دالة `request_review` ودالة `approve` ، فإنه يعيد نفسه لأن المنشور يجب أن يظل في حالة `Published` في تلك الحالات.

الآن نحتاج إلى تحديث دالة `content` في `Post`. نريد أن تعتمد القيمة المعادة من `content` على الحالة الحالية لـ `Post` ، لذا سنجعل `Post` يفوض (delegate) المهمة لدالة `content` معرفة في `state` الخاصة به، كما هو موضح في القائمة 18-17.

<Listing number="18-17" file-name="src/lib.rs" caption="تحديث دالة content في Post لتفويض المهمة لدالة content في State">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-17/src/lib.rs:here}}
```

</Listing>

بما أن الهدف هو إبقاء كل هذه القواعد داخل structs التي تنفذ `State` ، فإننا نستدعي دالة `content` على القيمة الموجودة في `state` ونمرر instance المنشور (أي `self`) كمعامل. ثم نعيد القيمة التي يتم إرجاعها من استخدام دالة `content` على قيمة `state`.

نستدعي دالة `as_ref` على `Option` لأننا نريد مرجعاً (reference) للقيمة الموجودة داخل `Option` بدلاً من ملكية القيمة. نظرًا لأن `state` هي من نوع `Option<Box<dyn State>>` ، فعندما نستدعي `as_ref` ، يتم إرجاع `Option<&Box<dyn State>>`. إذا لم نستدعِ `as_ref` ، فسنحصل على خطأ لأننا لا نستطيع نقل `state` خارج `&self` المستعار لمعامل الدالة.

ثم نستدعي دالة `unwrap` ، والتي نعلم أنها لن تتسبب أبداً في توقف طارئ (panic) لأننا نعلم أن methods في `Post` تضمن أن `state` ستحتوي دائماً على قيمة `Some` عند انتهاء تلك methods. هذه واحدة من الحالات التي تحدثنا عنها في قسم ["عندما تمتلك معلومات أكثر من المصرّف"][more-info-than-rustc] في الفصل التاسع عندما نعلم أن قيمة `None` غير ممكنة أبداً، على الرغم من أن compiler غير قادر على فهم ذلك.

عند هذه النقطة، عندما نستدعي `content` على `&Box<dyn State>` ، سيبدأ مفعول إكراه فك المرجع (deref coercion) على `&` و `Box` بحيث يتم استدعاء دالة `content` في النهاية على النوع الذي ينفذ `State` trait. وهذا يعني أننا بحاجة إلى إضافة `content` إلى تعريف `State` trait، وهنا سنضع المنطق الخاص بالمحتوى الذي سيتم إرجاعه بناءً على الحالة التي لدينا، كما هو موضح في القائمة 18-18.

<Listing number="18-18" file-name="src/lib.rs" caption="إضافة دالة content إلى سمة State">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-18/src/lib.rs:here}}
```

</Listing>

نضيف تنفيذاً افتراضياً (default implementation) لدالة `content` يعيد string slice فارغة. وهذا يعني أننا لسنا بحاجة لتنفيذ `content` في structs المسمى `Draft` و `PendingReview`. سيقوم struct المسمى `Published` بتجاوز (override) دالة `content` وإعادة القيمة الموجودة في `post.content`. على الرغم من كونه ملائماً، إلا أن جعل دالة `content` في `State` تحدد محتوى `Post` يؤدي إلى تداخل الحدود بين مسؤولية `State` ومسؤولية `Post`.

لاحظ أننا بحاجة إلى تعليقات توضيحية للعمر (lifetime annotations) في هذه الدالة، كما ناقشنا في الفصل العاشر. نحن نأخذ reference لـ `post` كمعامل ونعيد reference لجزء من ذلك `post` ، لذا فإن عمر المرجع المعاد مرتبط بعمر معامل `post`.

لقد انتهينا—كل ما في القائمة 18-11 يعمل الآن! لقد قمنا بتنفيذ state pattern مع قواعد سير عمل منشور المدونة. المنطق المتعلق بالقواعد يعيش في state objects بدلاً من أن يكون مشتتاً في جميع أنحاء `Post`.

> ### لماذا لا نستخدم التعداد (Enum)؟
>
> ربما كنت تتساءل لماذا لم نستخدم تعداداً (enum) مع حالات المنشور المختلفة كمتغيرات (variants). هذا بالتأكيد حل ممكن؛ جربه وقارن النتائج النهائية لترى أيهما تفضل! أحد عيوب استخدام enum هو أن كل مكان يتحقق من قيمة enum سيحتاج إلى تعبير مطابقة (match expression) أو ما شابه للتعامل مع كل variant ممكن. قد يصبح هذا أكثر تكراراً من حل trait object هذا.

<!-- Old headings. Do not remove or links may break. -->

<a id="trade-offs-of-the-state-pattern"></a>

#### تقييم نمط الحالة (Evaluating the State Pattern)

لقد أظهرنا أن Rust قادرة على تنفيذ object-oriented state pattern لتغليف (encapsulate) الأنواع المختلفة من السلوك التي يجب أن يتمتع بها المنشور في كل حالة. لا تعرف methods في `Post` شيئاً عن السلوكيات المختلفة. وبسبب الطريقة التي نظمنا بها الكود، علينا النظر في مكان واحد فقط لمعرفة الطرق المختلفة التي يمكن أن يتصرف بها المنشور المنشور: تنفيذ `State` trait على struct المسمى `Published`.

إذا أردنا إنشاء تنفيذ بديل لا يستخدم state pattern، فقد نستخدم بدلاً من ذلك `match` expressions في methods الخاصة بـ `Post` أو حتى في كود `main` الذي يتحقق من حالة المنشور ويغير السلوك في تلك الأماكن. وهذا يعني أننا سنضطر إلى النظر في عدة أماكن لفهم جميع تداعيات كون المنشور في حالة published.

مع state pattern، لا تحتاج methods في `Post` والأماكن التي نستخدم فيها `Post` إلى `match` expressions، ولإضافة حالة جديدة، سنحتاج فقط إلى إضافة struct جديد وتنفيذ trait methods على ذلك struct في مكان واحد.

التنفيذ باستخدام state pattern سهل التوسيع لإضافة المزيد من الوظائف. لرؤية بساطة صيانة الكود الذي يستخدم state pattern، جرب بعض هذه الاقتراحات:

- أضف دالة `reject` تغير حالة المنشور من `PendingReview` مرة أخرى إلى `Draft`.
- اشترط استدعاءين لـ `approve` قبل أن يتم تغيير الحالة إلى `Published`.
- اسمح للمستخدمين بإضافة محتوى نصي فقط عندما يكون المنشور في حالة `Draft`.
  تلميح: اجعل state object مسؤولاً عما قد يتغير في المحتوى ولكنه غير مسؤول عن تعديل `Post`.

أحد عيوب state pattern هو أنه نظراً لأن الحالات تنفذ الانتقالات بين الحالات، فإن بعض الحالات مرتبطة (coupled) ببعضها البعض. إذا أضفنا حالة أخرى بين `PendingReview` و `Published` ، مثل `Scheduled` ، فسنضطر إلى تغيير الكود في `PendingReview` للانتقال إلى `Scheduled` بدلاً من ذلك. سيكون العمل أقل إذا لم تكن `PendingReview` بحاجة إلى التغيير مع إضافة حالة جديدة، ولكن هذا يعني الانتقال إلى نمط تصميم آخر.

عيب آخر هو أننا قمنا بتكرار بعض المنطق. للتخلص من بعض التكرار، قد نحاول عمل default implementations لدوال `request_review` و `approve` في `State` trait تعيد `self`. ومع ذلك، لن يعمل هذا: عند استخدام `State` كـ trait object، لا تعرف trait ما سيكون عليه `self` الملموس (concrete) بالضبط، لذا فإن نوع الإرجاع غير معروف في وقت التصريف. (هذه واحدة من قواعد توافق dyn المذكورة سابقاً.)

يتضمن التكرار الآخر التنفيذات المتشابهة لدوال `request_review` و `approve` في `Post`. تستخدم كلتا الدالتين `Option::take` مع حقل `state` في `Post` ، وإذا كانت `state` هي `Some` ، فإنهما تفوضان المهمة لتنفيذ القيمة المغلفة لنفس الدالة وتضبطان القيمة الجديدة لحقل `state` على النتيجة. إذا كان لدينا الكثير من methods في `Post` تتبع هذا النمط، فقد نفكر في تعريف ماكرو (macro) للتخلص من التكرار (انظر قسم ["الماكرو"][macros] في الفصل 20).

من خلال تنفيذ state pattern تماماً كما هو محدد للغات كائنية التوجه، فإننا لا نستفيد بشكل كامل من نقاط قوة Rust كما يمكننا. دعونا نلقي نظرة على بعض التغييرات التي يمكننا إجراؤها على crate المسمى `blog` والتي يمكن أن تجعل الحالات والانتقالات غير الصالحة أخطاء في وقت التصريف (compile-time errors).

### ترميز الحالات والسلوك كأنواع (Encoding States and Behavior as Types)

سنوضح لك كيفية إعادة التفكير في state pattern للحصول على مجموعة مختلفة من trade-offs. بدلاً من تغليف الحالات والانتقالات تماماً بحيث لا يكون للكود الخارجي أي معرفة بها، سنقوم بترميز (encode) الحالات في أنواع مختلفة. وبالتالي، سيمنع نظام فحص الأنواع (type-checking system) في Rust محاولات استخدام منشورات draft حيث يُسمح فقط بمنشورات published عن طريق إصدار خطأ من compiler.

دعونا نفكر في الجزء الأول من `main` في القائمة 18-11:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-11/src/main.rs:here}}
```

</Listing>

لا نزال نمكن من إنشاء منشورات جديدة في حالة draft باستخدام `Post::new` والقدرة على إضافة نص إلى محتوى المنشور. ولكن بدلاً من وجود دالة `content` في منشور draft تعيد سلسلة نصية فارغة، سنجعل الأمر بحيث لا تمتلك منشورات draft دالة `content` على الإطلاق. بهذه الطريقة، إذا حاولنا الحصول على محتوى منشور draft، فسنحصل على خطأ من compiler يخبرنا أن الدالة غير موجودة. ونتيجة لذلك، سيكون من المستحيل بالنسبة لنا عرض محتوى منشور draft عن طريق الخطأ في الإنتاج لأن ذلك الكود لن يتم تصريفه أصلاً. توضح القائمة 18-19 تعريف struct المسمى `Post` و struct المسمى `DraftPost` ، بالإضافة إلى methods في كل منهما.

<Listing number="18-19" file-name="src/lib.rs" caption="هيكل Post مع دالة content وهيكل DraftPost بدون دالة content">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-19/src/lib.rs}}
```

</Listing>

يمتلك كل من structs المسمى `Post` و `DraftPost` حقل `content` خاصاً يخزن نص منشور المدونة. لم تعد structs تمتلك حقل `state` لأننا ننقل ترميز الحالة إلى أنواع structs. سيمثل struct المسمى `Post` منشوراً منشوراً، ولديه دالة `content` تعيد `content`.

لا يزال لدينا دالة `Post::new` ، ولكن بدلاً من إرجاع instance من `Post` ، فإنها تعيد instance من `DraftPost`. نظرًا لأن `content` خاص ولا توجد أي functions تعيد `Post` ، فليس من الممكن إنشاء instance من `Post` في الوقت الحالي.

يمتلك struct المسمى `DraftPost` دالة `add_text` ، لذا يمكننا إضافة نص إلى `content` كما كان من قبل، ولكن لاحظ أن `DraftPost` لا يمتلك دالة `content` معرفة! لذا الآن يضمن البرنامج أن جميع المنشورات تبدأ كمنشورات draft، ومنشورات draft ليس محتواها متاحاً للعرض. أي محاولة للحصول على
محتوى هذه القيود سيؤدي إلى خطأ من compiler.

إذن، كيف نحصل على منشور منشور؟ نريد فرض القاعدة التي تنص على أن منشور draft يجب مراجعته والموافقة عليه قبل أن يتم نشره. يجب ألا يعرض المنشور في حالة pending review أي محتوى أيضاً. دعونا ننفذ هذه القيود عن طريق إضافة struct آخر يسمى `PendingReviewPost` ، وتعريف دالة `request_review` في `DraftPost` لتعيد `PendingReviewPost` وتعريف دالة `approve` في `PendingReviewPost` لتعيد `Post` ، كما هو موضح في القائمة 18-20.

<Listing number="18-20" file-name="src/lib.rs" caption="هيكل PendingReviewPost الذي يتم إنشاؤه عن طريق استدعاء request_review على DraftPost ودالة approve التي تحول PendingReviewPost إلى Post منشور">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-20/src/lib.rs:here}}
```

</Listing>

تأخذ دالتا `request_review` و `approve` ملكية `self` ، وبالتالي تستهلكان مثيلات `DraftPost` و `PendingReviewPost` وتحولانهما إلى `PendingReviewPost` و `Post` منشور، على التوالي. بهذه الطريقة، لن يكون لدينا أي مثيلات `DraftPost` متبقية بعد استدعاء `request_review` عليها، وهكذا دواليك. لا يمتلك struct المسمى `PendingReviewPost` دالة `content` معرفة فيه، لذا فإن محاولة قراءة محتواه تؤدي إلى خطأ من compiler، كما هو الحال مع `DraftPost`. ولأن الطريقة الوحيدة للحصول على instance من `Post` منشور يمتلك دالة `content` معرفة هي استدعاء دالة `approve` على `PendingReviewPost` ، والطريقة الوحيدة للحصول على `PendingReviewPost` هي استدعاء دالة `request_review` على `DraftPost` ، فقد قمنا الآن بترميز سير عمل منشور المدونة في نظام الأنواع (type system).

ولكن علينا أيضاً إجراء بعض التغييرات الصغيرة في `main`. تعيد دالتا `request_review` و `approve` مثيلات جديدة بدلاً من تعديل struct الذي يتم استدعاؤهما عليه، لذا نحتاج إلى إضافة المزيد من تعيينات التظليل (shadowing assignments) باستخدام `let post =` لحفظ المثيلات المعادة. كما لا يمكننا إجراء التأكيدات (assertions) حول كون محتويات منشورات draft و pending review سلاسل نصية فارغة، ولسنا بحاجة إليها: لا يمكننا تصريف الكود الذي يحاول استخدام محتوى المنشورات في تلك الحالات بعد الآن. يظهر الكود المحدث في `main` في القائمة 18-21.

<Listing number="18-21" file-name="src/main.rs" caption="تعديلات على main لاستخدام التنفيذ الجديد لسير عمل منشور المدونة">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-21/src/main.rs}}
```

</Listing>

التغييرات التي احتجنا لإجرائها في `main` لإعادة تعيين `post` تعني أن هذا التنفيذ لا يتبع تماماً object-oriented state pattern بعد الآن: التحويلات بين الحالات لم تعد مغلفة بالكامل داخل تنفيذ `Post`. ومع ذلك، فإن مكسبنا هو أن الحالات غير الصالحة أصبحت الآن مستحيلة بسبب نظام الأنواع وفحص الأنواع الذي يحدث في وقت التصريف! يضمن هذا اكتشاف بعض الأخطاء، مثل عرض محتوى منشور غير منشور، قبل وصولها إلى الإنتاج.

جرب المهام المقترحة في بداية هذا القسم على crate المسمى `blog` كما هو بعد القائمة 18-21 لترى رأيك في تصميم هذا الإصدار من الكود. لاحظ أن بعض المهام قد تكون مكتملة بالفعل في هذا التصميم.

لقد رأينا أنه على الرغم من أن Rust قادرة على تنفيذ أنماط التصميم كائنية التوجه، إلا أن أنماطاً أخرى، مثل ترميز الحالة في نظام الأنواع، متاحة أيضاً في Rust. هذه الأنماط لها trade-offs مختلفة. على الرغم من أنك قد تكون على دراية كبيرة بالأنماط كائنية التوجه، إلا أن إعادة التفكير في المشكلة للاستفادة من ميزات Rust يمكن أن توفر فوائد، مثل منع بعض الأخطاء في وقت التصريف. لن تكون الأنماط كائنية التوجه دائماً هي الحل الأفضل في Rust بسبب ميزات معينة، مثل الملكية (ownership)، التي لا تمتلكها اللغات كائنية التوجه.

## ملخص (Summary)

بغض النظر عما إذا كنت تعتقد أن Rust لغة كائنية التوجه بعد قراءة هذا الفصل، فأنت تعلم الآن أنه يمكنك استخدام trait objects للحصول على بعض الميزات كائنية التوجه في Rust. يمكن أن يمنح الإرسال الديناميكي (dynamic dispatch) كودك بعض المرونة مقابل القليل من أداء وقت التشغيل. يمكنك استخدام هذه المرونة لتنفيذ أنماط كائنية التوجه يمكن أن تساعد في قابلية صيانة كودك. تمتلك Rust أيضاً ميزات أخرى، مثل ownership، لا تمتلكها اللغات كائنية التوجه. لن يكون النمط كائني التوجه دائماً هو أفضل طريقة للاستفادة من نقاط قوة Rust، ولكنه خيار متاح.

بعد ذلك، سننظر في الأنماط (patterns)، وهي ميزة أخرى من ميزات Rust التي تتيح الكثير من المرونة. لقد نظرنا إليها باختصار طوال الكتاب ولكننا لم نرَ كامل قدراتها بعد. لننطلق!

[more-info-than-rustc]: ch09-03-to-panic-or-not-to-panic.html#cases-in-which-you-have-more-information-than-the-compiler
[macros]: ch20-05-macros.html#macros
