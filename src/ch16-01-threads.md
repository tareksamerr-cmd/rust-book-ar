## استخدام الخيوط لتشغيل الكود في وقت واحد (Using Threads to Run Code Simultaneously)

في معظم أنظمة التشغيل الحالية، يتم تشغيل كود البرنامج المنفذ في _عملية_ (process)، وسيقوم نظام التشغيل بإدارة عدة processes في وقت واحد. داخل البرنامج، يمكنك أيضاً الحصول على أجزاء مستقلة تعمل في وقت واحد. تسمى الميزات التي تشغل هذه الأجزاء المستقلة _خيوطاً_ (threads). على سبيل المثال، يمكن أن يحتوي خادم الويب على عدة threads بحيث يمكنه الاستجابة لأكثر من طلب واحد في نفس الوقت.

يمكن أن يؤدي تقسيم الحساب في برنامجك إلى عدة threads لتشغيل مهام متعددة في نفس الوقت إلى تحسين الأداء، ولكنه يضيف أيضاً تعقيداً. ولأن threads يمكن أن تعمل في وقت واحد، فلا يوجد ضمان متأصل حول الترتيب الذي ستعمل به أجزاء الكود الخاصة بك على threads مختلفة. يمكن أن يؤدي هذا إلى مشاكل، مثل:

- حالات السباق (Race conditions)، حيث تصل threads إلى البيانات أو الموارد بترتيب غير متسق.
- حالات الجمود (Deadlocks)، حيث ينتظر خيطان بعضهما البعض، مما يمنع كلا الـ threads من الاستمرار.
- الأخطاء (Bugs) التي تحدث فقط في مواقف معينة ويصعب إعادة إنتاجها وإصلاحها بشكل موثوق.

تحاول Rust التخفيف من الآثار السلبية لاستخدام threads، لكن البرمجة في سياق متعدد الخيوط (multithreaded) لا تزال تتطلب تفكيراً دقيقاً وتتطلب بنية كود تختلف عن تلك الموجودة في البرامج التي تعمل في خيط واحد (single thread).

تنفذ لغات البرمجة threads بعدة طرق مختلفة، وتوفر العديد من أنظمة التشغيل واجهة برمجة تطبيقات (API) يمكن للغة البرمجة استدعاؤها لإنشاء threads جديدة. تستخدم مكتبة Rust القياسية نموذج _1:1_ لتنفيذ الخيوط، حيث يستخدم البرنامج خيط نظام تشغيل واحداً لكل خيط لغة واحد. هناك صناديق (crates) تنفذ نماذج أخرى من threading تقدم مقايضات مختلفة لنموذج 1:1. (يوفر نظام async في Rust، والذي سنراه في الفصل القادم، نهجاً آخر للتزامن (concurrency) أيضاً.)

### إنشاء خيط جديد باستخدام `spawn` (Creating a New Thread with `spawn`)

لإنشاء خيط جديد، نستدعي دالة `thread::spawn` ونمرر لها إغلاقاً (closure) (تحدثنا عن closures في الفصل الثالث عشر) يحتوي على الكود الذي نريد تشغيله في الخيط الجديد. يطبع المثال في القائمة 16-1 بعض النصوص من خيط رئيسي (main thread) ونصوصاً أخرى من خيط جديد.

<Listing number="16-1" file-name="src/main.rs" caption="Creating a new thread to print one thing while the main thread prints something else">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-01/src/main.rs}}
```

</Listing>

لاحظ أنه عندما يكتمل main thread لبرنامج Rust، يتم إغلاق جميع الـ spawned threads، سواء انتهت من العمل أم لا. قد يكون المخرجات من هذا البرنامج مختلفة قليلاً في كل مرة، لكنها ستشبه ما يلي:

```text
hi number 1 from the main thread!
hi number 1 from the spawned thread!
hi number 2 from the main thread!
hi number 2 from the spawned thread!
hi number 3 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the main thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
```

تجبر استدعاءات `thread::sleep` الخيط على إيقاف تنفيذه لفترة قصيرة، مما يسمح لخيط مختلف بالعمل. من المحتمل أن تتبادل الـ threads الأدوار، لكن هذا ليس مضموناً: فهو يعتمد على كيفية جدولة نظام التشغيل الخاص بك للـ threads. في هذا التشغيل، طبع main thread أولاً، على الرغم من أن جملة الطباعة من spawned thread تظهر أولاً في الكود. وحتى بالرغم من أننا أخبرنا spawned thread بالطباعة حتى تصل `i` إلى `9` ، إلا أنه وصل فقط إلى `5` قبل أن يتوقف main thread.

إذا قمت بتشغيل هذا الكود ورأيت فقط مخرجات من main thread، أو لم ترَ أي تداخل، فحاول زيادة الأرقام في النطاقات لإنشاء المزيد من الفرص لنظام التشغيل للتبديل بين الـ threads.

<!-- Old headings. Do not remove or links may break. -->

<a id="waiting-for-all-threads-to-finish-using-join-handles"></a>

### انتظار انتهاء جميع الخيوط (Waiting for All Threads to Finish)

الكود في القائمة 16-1 لا يوقف spawned thread قبل الأوان في معظم الأوقات بسبب انتهاء main thread فحسب، بل لأنه لا يوجد ضمان على الترتيب الذي تعمل به الـ threads، لا يمكننا أيضاً ضمان أن spawned thread سيعمل على الإطلاق!

يمكننا إصلاح مشكلة عدم عمل spawned thread أو انتهائه قبل الأوان عن طريق حفظ قيمة الإرجاع لـ `thread::spawn` في متغير. نوع الإرجاع لـ `thread::spawn` هو `JoinHandle<T>`. الـ `JoinHandle<T>` هو قيمة مملوكة، عندما نستدعي طريقة (method) الـ `join` عليها، ستنتظر خيطها حتى ينتهي. توضح القائمة 16-2 كيفية استخدام `JoinHandle<T>` للخيط الذي أنشأناه في القائمة 16-1 وكيفية استدعاء `join` للتأكد من انتهاء spawned thread قبل خروج `main`.

<Listing number="16-2" file-name="src/main.rs" caption="Saving a `JoinHandle<T>` from `thread::spawn` to guarantee the thread is run to completion">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-02/src/main.rs}}
```

</Listing>

يؤدي استدعاء `join` على المقبض (handle) إلى حظر (block) الخيط الذي يعمل حالياً حتى ينتهي الخيط الذي يمثله الـ handle. _حظر_ (Blocking) الخيط يعني منع ذلك الخيط من أداء العمل أو الخروج. ولأننا وضعنا استدعاء `join` بعد حلقة `for` الخاصة بـ main thread، فإن تشغيل القائمة 16-2 يجب أن ينتج مخرجات مشابهة لهذا:

```text
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 1 from the spawned thread!
hi number 3 from the main thread!
hi number 2 from the spawned thread!
hi number 4 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
```

يستمر الخيطان في التناوب، لكن main thread ينتظر بسبب استدعاء `handle.join()` ولا ينتهي حتى ينتهي spawned thread.

ولكن دعونا نرى ما يحدث عندما نقوم بدلاً من ذلك بنقل `handle.join()` قبل حلقة `for` في `main` ، هكذا:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/no-listing-01-join-too-early/src/main.rs}}
```

</Listing>

سينتظر main thread انتهاء spawned thread ثم يشغل حلقة `for` الخاصة به، لذا لن تكون المخرجات متداخلة بعد الآن، كما هو موضح هنا:

```text
hi number 1 from the spawned thread!
hi number 2 from the spawned thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 3 from the main thread!
hi number 4 from the main thread!
```

يمكن للتفاصيل الصغيرة، مثل مكان استدعاء `join` ، أن تؤثر على ما إذا كانت الـ threads تعمل في نفس الوقت أم لا.

### استخدام إغلاقات `move` مع الخيوط (Using `move` Closures with Threads)

سنستخدم غالباً الكلمة المفتاحية `move` مع closures الممررة إلى `thread::spawn` لأن الـ closure سيأخذ حينها ملكية (ownership) القيم التي يستخدمها من البيئة، وبالتالي ينقل ملكية تلك القيم من خيط إلى آخر. في قسم ["التقاط المراجع أو نقل الملكية"][capture] في الفصل الثالث عشر، ناقشنا `move` في سياق closures. الآن سنركز أكثر على التفاعل بين `move` و `thread::spawn`.

لاحظ في القائمة 16-1 أن الـ closure الذي نمرره إلى `thread::spawn` لا يأخذ أي arguments: نحن لا نستخدم أي بيانات من main thread في كود spawned thread. لاستخدام بيانات من main thread في spawned thread، يجب على closure الخاص بـ spawned thread التقاط (capture) القيم التي يحتاجها. توضح القائمة 16-3 محاولة لإنشاء متجه (vector) في main thread واستخدامه في spawned thread. ومع ذلك، لن يعمل هذا بعد، كما سترى في لحظة.

<Listing number="16-3" file-name="src/main.rs" caption="Attempting to use a vector created by the main thread in another thread">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-03/src/main.rs}}
```

</Listing>

يستخدم الـ closure المتغير `v` ، لذا سيقوم بـ capture لـ `v` ويجعله جزءاً من بيئة الـ closure. ولأن `thread::spawn` يشغل هذا الـ closure في خيط جديد، يجب أن نكون قادرين على الوصول إلى `v` داخل ذلك الخيط الجديد. ولكن عندما نقوم بتجميع هذا المثال، نحصل على الخطأ التالي:

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-03/output.txt}}
```

تستنتج (infers) Rust كيفية التقاط `v` ، ولأن `println!` تحتاج فقط إلى مرجع (reference) لـ `v` ، يحاول الـ closure استعارة (borrow) المتغير `v`. ومع ذلك، هناك مشكلة: لا تستطيع Rust معرفة مدة تشغيل spawned thread، لذا فهي لا تعرف ما إذا كان الـ reference لـ `v` سيكون صالحاً دائماً.

تقدم القائمة 16-4 سيناريو من المرجح أن يحتوي على reference لـ `v` لن يكون صالحاً.

<Listing number="16-4" file-name="src/main.rs" caption="A thread with a closure that attempts to capture a reference to `v` from a main thread that drops `v`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-04/src/main.rs}}
```

</Listing>

إذا سمحت لنا Rust بتشغيل هذا الكود، فهناك احتمال أن يتم وضع spawned thread فوراً في الخلفية دون أن يعمل على الإطلاق. يحتوي spawned thread على reference لـ `v` بالداخل، لكن main thread يقوم فوراً بإسقاط (drop) المتغير `v` ، باستخدام دالة `drop` التي ناقشناها في الفصل الخامس عشر. ثم، عندما يبدأ spawned thread في التنفيذ، لن يكون `v` صالحاً بعد الآن، لذا فإن الـ reference له يكون أيضاً غير صالح. أوه لا!

لإصلاح خطأ المترجم (compiler) في القائمة 16-3، يمكننا استخدام نصيحة رسالة الخطأ:

```text
help: to force the closure to take ownership of `v` (and any other referenced variables), use the `move` keyword
  |
6 |     let handle = thread::spawn(move || {
  |                                ++++
```

بإضافة الكلمة المفتاحية `move` قبل الـ closure، نجبر الـ closure على أخذ ownership للقيم التي يستخدمها بدلاً من السماح لـ Rust باستنتاج أنه يجب عليه borrow للقيم. التعديل على القائمة 16-3 الموضح في القائمة 16-5 سيتم تجميعه وتشغيله كما ننوي.

<Listing number="16-5" file-name="src/main.rs" caption="Using the `move` keyword to force a closure to take ownership of the values it uses">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-05/src/main.rs}}
```

</Listing>

قد نميل إلى تجربة نفس الشيء لإصلاح الكود في القائمة 16-4 حيث استدعى main thread الدالة `drop` باستخدام `move` closure. ومع ذلك، فإن هذا الإصلاح لن يعمل لأن ما تحاول القائمة 16-4 القيام به غير مسموح به لسبب مختلف. إذا أضفنا `move` إلى الـ closure، فسننقل `v` إلى بيئة الـ closure، ولن نتمكن بعد الآن من استدعاء `drop` عليه في main thread. سنحصل على خطأ compiler هذا بدلاً من ذلك:

```console
{{#include ../listings/ch16-fearless-concurrency/output-only-01-move-drop/output.txt}}
```

لقد أنقذتنا قواعد الملكية (ownership rules) في Rust مرة أخرى! حصلنا على خطأ من الكود في القائمة 16-3 لأن Rust كانت متحفظة وتقوم فقط بـ borrow لـ `v` للخيط، مما يعني أن main thread يمكنه نظرياً إبطال reference الخاص بـ spawned thread. من خلال إخبار Rust بنقل ownership لـ `v` إلى spawned thread، فإننا نضمن لـ Rust أن main thread لن يستخدم `v` بعد الآن. إذا قمنا بتغيير القائمة 16-4 بنفس الطريقة، فإننا ننتهك ownership rules عندما نحاول استخدام `v` في main thread. الكلمة المفتاحية `move` تتجاوز افتراض Rust المتحفظ بالاستعارة؛ فهي لا تسمح لنا بانتهاك ownership rules.

الآن بعد أن غطينا ماهية الـ threads والطرق التي توفرها thread API، دعونا نلقي نظرة على بعض المواقف التي يمكننا فيها استخدام threads.

[capture]: ch13-01-closures.html#capturing-references-or-moving-ownership
```
