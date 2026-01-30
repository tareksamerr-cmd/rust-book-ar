## المتغيرات والقابلية للتغير (Variables and Mutability)

كما ذُكر في قسم ["تخزين القيم باستخدام المتغيرات"][storing-values-with-variables]<!-- ignore -->، تكون المتغيرات (Variables) غير قابلة للتغير (Immutable) بشكل افتراضي. هذه واحدة من العديد من التوجيهات التي تقدمها لك لغة Rust لكتابة الكود (Code) الخاص بك بطريقة تستفيد من الأمان والتزامن (Concurrency) السهل الذي توفره Rust. ومع ذلك، لا يزال لديك الخيار لجعل Variables الخاصة بك قابلة للتغير (Mutable). دعنا نستكشف كيف ولماذا تشجعك Rust على تفضيل عدم القابلية للتغير (Immutability) ولماذا قد ترغب أحياناً في إلغاء هذا الاختيار.

عندما يكون Variable غير قابل للتغير، فبمجرد ربط قيمة باسم ما، لا يمكنك تغيير تلك القيمة. لتوضيح ذلك، قم بإنشاء مشروع جديد يسمى _variables_ في دليل المشاريع الخاص بك باستخدام الأمر `cargo new variables`.

بعد ذلك، في دليل _variables_ الجديد، افتح ملف _src/main.rs_ واستبدل الكود الموجود فيه بالكود التالي، والذي لن يتم تحويله برمجياً (Compile) بعد:

<span class="filename">اسم الملف: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-01-variables-are-immutable/src/main.rs}}
```

احفظ البرنامج وشغله باستخدام `cargo run`. يجب أن تتلقى رسالة خطأ تتعلق بخطأ في Immutability، كما هو موضح في هذا المخرج:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-01-variables-are-immutable/output.txt}}
```

يوضح هذا المثال كيف يساعدك المترجم (Compiler) في العثور على الأخطاء في برامجك. قد تكون أخطاء Compiler محبطة، لكنها في الحقيقة تعني فقط أن برنامجك لا يفعل ما تريده بأمان بعد؛ وهي _لا_ تعني أنك لست مبرمجاً جيداً! حتى مبرمجي Rust المتمرسين لا يزالون يتلقون أخطاء Compiler.

لقد تلقيت رسالة الخطأ `` cannot assign twice to immutable variable `x` `` لأنك حاولت تعيين قيمة ثانية لـ Variable `x` غير القابل للتغير.

من المهم أن نحصل على أخطاء في وقت التحويل البرمجي (Compile-time Errors) عندما نحاول تغيير قيمة تم تحديدها على أنها Immutable، لأن هذا الموقف بحد ذاته يمكن أن يؤدي إلى أخطاء برمجية (Bugs). إذا كان جزء من Code الخاص بنا يعمل بناءً على افتراض أن القيمة لن تتغير أبداً وقام جزء آخر من Code بتغيير تلك القيمة، فمن المحتمل ألا يقوم الجزء الأول من Code بما صُمم للقيام به. قد يكون من الصعب تتبع سبب هذا النوع من Bug بعد وقوعه، خاصة عندما يغير الجزء الثاني من Code القيمة _أحياناً_ فقط. يضمن Compiler الخاص بـ Rust أنه عندما تصرح بأن القيمة لن تتغير، فإنها لن تتغير حقاً، لذا لا يتعين عليك تتبع ذلك بنفسك. وبالتالي يصبح من الأسهل فهم منطق Code الخاص بك.

لكن القابلية للتغير (Mutability) يمكن أن تكون مفيدة جداً ويمكن أن تجعل كتابة Code أكثر ملاءمة. على الرغم من أن Variables تكون Immutable افتراضياً، يمكنك جعلها Mutable عن طريق إضافة الكلمة المفتاحية `mut` أمام اسم Variable كما فعلت في [الفصل 2][storing-values-with-variables]<!-- ignore -->. إضافة `mut` تنقل أيضاً النية للقراء المستقبليين لـ Code من خلال الإشارة إلى أن أجزاء أخرى من Code ستغير قيمة Variable هذا.

على سبيل المثال، لنقم بتغيير _src/main.rs_ إلى ما يلي:

<span class="filename">اسم الملف: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-02-adding-mut/src/main.rs}}
```

عندما نشغل البرنامج الآن، نحصل على هذا:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-02-adding-mut/output.txt}}
```

يُسمح لنا بتغيير القيمة المرتبطة بـ `x` من `5` إلى `6` عند استخدام `mut`. في النهاية، قرار استخدام Mutability من عدمه يعود إليك ويعتمد على ما تعتقد أنه الأكثر وضوحاً في ذلك الموقف المعين.

<!-- العناوين القديمة. لا تقم بإزالتها وإلا قد تتعطل الروابط. -->
<a id="constants"></a>

### التصريح عن الثوابت (Declaring Constants)

مثل Variables غير القابلة للتغير، فإن _الثوابت_ (Constants) هي قيم مرتبطة باسم ولا يُسمح بتغييرها، ولكن هناك بعض الاختلافات بين Constants و Variables.

أولاً، لا يُسمح لك باستخدام `mut` مع Constants. الثوابت ليست فقط Immutable افتراضياً - بل هي دائماً Immutable. تقوم بالتصريح عن Constants باستخدام الكلمة المفتاحية `const` بدلاً من الكلمة المفتاحية `let` ، و _يجب_ تحديد نوع (Type) القيمة. سنغطي الأنواع وتوصيفات الأنواع (Type Annotations) في القسم التالي، ["أنواع البيانات"][data-types]<!-- ignore -->، لذا لا تقلق بشأن التفاصيل الآن. فقط اعلم أنه يجب عليك دائماً توصيف Type.

يمكن التصريح عن Constants في أي نطاق (Scope)، بما في ذلك Scope العالمي، مما يجعلها مفيدة للقيم التي تحتاج أجزاء كثيرة من Code إلى معرفتها.

الاختلاف الأخير هو أنه لا يجوز تعيين Constants إلا لتعبير ثابت (Constant Expression)، وليس لنتيجة قيمة لا يمكن حسابها إلا في وقت التشغيل (Runtime).

إليك مثال على التصريح عن ثابت:

```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

اسم الثابت هو `THREE_HOURS_IN_SECONDS` ، وقيمته محددة بنتيجة ضرب 60 (عدد الثواني في الدقيقة) في 60 (عدد الدقائق في الساعة) في 3 (عدد الساعات التي نريد عدها في هذا البرنامج). اصطلاح التسمية في Rust لـ Constants هو استخدام جميع الأحرف الكبيرة مع وجود شرطات سفلية بين الكلمات. يستطيع Compiler تقييم مجموعة محدودة من العمليات في وقت التحويل البرمجي، مما يتيح لنا اختيار كتابة هذه القيمة بطريقة يسهل فهمها والتحقق منها، بدلاً من تعيين هذا الثابت للقيمة 10,800. راجع [قسم مرجع Rust حول تقييم الثوابت][const-eval] لمزيد من المعلومات حول العمليات التي يمكن استخدامها عند التصريح عن Constants.

تكون Constants صالحة طوال وقت تشغيل البرنامج، ضمن Scope الذي تم التصريح عنها فيه. تجعل هذه الخاصية Constants مفيدة للقيم في مجال تطبيقك التي قد تحتاج أجزاء متعددة من البرنامج إلى معرفتها، مثل الحد الأقصى لعدد النقاط التي يُسمح لأي لاعب في اللعبة بالحصول عليها، أو سرعة الضوء.

يعد تسمية القيم المكتوبة مباشرة (Hardcoded Values) المستخدمة في جميع أنحاء برنامجك كـ Constants أمراً مفيداً في نقل معنى تلك القيمة للمسؤولين عن صيانة Code في المستقبل. يساعد أيضاً وجود مكان واحد فقط في Code الخاص بك تحتاج إلى تغييره إذا احتاجت Hardcoded Value إلى التحديث في المستقبل.

### الحجب (Shadowing)

كما رأيت في برنامج لعبة التخمين في [الفصل 2][comparing-the-guess-to-the-secret-number]<!-- ignore -->، يمكنك التصريح عن Variable جديد بنفس اسم Variable سابق. يقول مبرمجو Rust إن Variable الأول قد تم _حجبه_ (Shadowed) بواسطة الثاني، مما يعني أن Variable الثاني هو ما سيراه Compiler عند استخدام اسم Variable. في الواقع، يحجب Variable الثاني الأول، ويأخذ أي استخدامات لاسم Variable لنفسه حتى يتم حجبه هو نفسه أو ينتهي Scope. يمكننا حجب Variable باستخدام نفس اسم Variable وتكرار استخدام الكلمة المفتاحية `let` كما يلي:

<span class="filename">اسم الملف: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-03-shadowing/src/main.rs}}
```

يربط هذا البرنامج أولاً `x` بقيمة `5`. ثم ينشئ Variable جديد `x` بتكرار `let x =` ، آخذاً القيمة الأصلية ومضيفاً `1` بحيث تصبح قيمة `x` هي `6`. ثم، داخل Scope داخلي تم إنشاؤه باستخدام الأقواس المتعرجة، تحجب جملة `let` الثالثة أيضاً `x` وتنشئ Variable جديداً، وتضرب القيمة السابقة في `2` لتعطي `x` قيمة `12`. عندما ينتهي هذا Scope، ينتهي الحجب الداخلي ويعود `x` ليكون `6`. عندما نشغل هذا البرنامج، سيخرج ما يلي:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-03-shadowing/output.txt}}
```

يختلف Shadowing عن تمييز Variable كـ `mut` لأننا سنحصل على Compile-time Error إذا حاولنا بالخطأ إعادة التعيين لهذا Variable دون استخدام الكلمة المفتاحية `let`. باستخدام `let` ، يمكننا إجراء بعض التحويلات على قيمة ولكن يظل Variable غير قابل للتغير بعد اكتمال تلك التحويلات.

الفرق الآخر بين `mut` و Shadowing هو أننا نظراً لأننا نقوم فعلياً بإنشاء Variable جديد عندما نستخدم الكلمة المفتاحية `let` مرة أخرى، يمكننا تغيير Type القيمة ولكن مع إعادة استخدام نفس الاسم. على سبيل المثال، لنفترض أن برنامجنا يطلب من المستخدم إظهار عدد المسافات التي يريدها بين بعض النصوص عن طريق إدخال أحرف مسافة، ثم نريد تخزين هذا الإدخال كرقم:

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-04-shadowing-can-change-types/src/main.rs:here}}
```

أول Variable باسم `spaces` هو من نوع سلسلة نصية (String Type)، وثاني Variable باسم `spaces` هو من نوع رقمي (Number Type). وبالتالي يجنبنا Shadowing الاضطرار إلى ابتكار أسماء مختلفة، مثل `spaces_str` و `spaces_num`؛ بدلاً من ذلك، يمكننا إعادة استخدام اسم `spaces` الأبسط. ومع ذلك، إذا حاولنا استخدام `mut` لهذا، كما هو موضح هنا، فسنحصل على Compile-time Error:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-05-mut-cant-change-types/src/main.rs:here}}
```

يقول الخطأ إنه لا يُسمح لنا بتغيير نوع (Mutate a variable's type) المتغير:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-05-mut-cant-change-types/output.txt}}
```

الآن بعد أن استكشفنا كيفية عمل Variables، دعنا نلقي نظرة على المزيد من أنواع البيانات (Data Types) التي يمكن أن تمتلكها.

[comparing-the-guess-to-the-secret-number]: ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[data-types]: ch03-02-data-types.html#data-types
[storing-values-with-variables]: ch02-00-guessing-game-tutorial.html#storing-values-with-variables
[const-eval]: ../reference/const_eval.html
