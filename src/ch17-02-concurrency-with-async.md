<!-- Old headings. Do not remove or links may break. -->

<a id="concurrency-with-async"></a>

## تطبيق التزامن باستخدام Async (Applying Concurrency with Async)

في هذا القسم، سنقوم بتطبيق async على بعض تحديات التزامن (concurrency) نفسها التي واجهناها مع الخيوط (threads) في الفصل 16. نظرًا لأننا تحدثنا بالفعل عن الكثير من الأفكار الرئيسية هناك، سنركز في هذا القسم على الاختلافات بين الـ threads والعقود الآجلة (futures).

في كثير من الحالات، تكون واجهات برمجة التطبيقات (APIs) للعمل مع التزامن باستخدام async مشابهة جدًا لتلك المستخدمة مع الـ threads. وفي حالات أخرى، ينتهي بها الأمر لتكون مختلفة تمامًا. وحتى عندما _تبدو_ الـ APIs متشابهة بين الـ threads و async، فغالبًا ما يكون لها سلوك مختلف - ودائمًا ما يكون لها خصائص أداء مختلفة.

<!-- Old headings. Do not remove or links may break. -->

<a id="counting"></a>

### إنشاء مهمة جديدة باستخدام `spawn_task` (Creating a New Task with `spawn_task`)

كانت العملية الأولى التي تناولناها في قسم ["إنشاء خيط جديد باستخدام `spawn`"][thread-spawn]<!-- ignore --> في الفصل 16 هي العد التصاعدي في خيطين منفصلين. لنفعل الشيء نفسه باستخدام async. توفر حزمة `trpl` دالة `spawn_task` التي تبدو مشابهة جدًا لـ API `thread::spawn` ودالة `sleep` التي تعد نسخة async من API `thread::sleep`. يمكننا استخدامهما معًا لتنفيذ مثال العد، كما هو موضح في القائمة 17-6.

<Listing number="17-6" caption="إنشاء مهمة جديدة لطباعة شيء واحد بينما تطبع المهمة الرئيسية شيئًا آخر" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-06/src/main.rs:all}}
```

</Listing>

كنقطة بداية، نقوم بإعداد دالة `main` الخاصة بنا باستخدام `trpl::block_on` بحيث يمكن أن تكون دالتنا ذات المستوى الأعلى async.

> ملاحظة: من هذه النقطة فصاعدًا في الفصل، سيتضمن كل مثال نفس كود التغليف هذا مع `trpl::block_on` في `main` لذا سنقوم غالبًا بتخطيه تمامًا كما نفعل مع `main`. تذكر تضمينه في كودك!

ثم نكتب حلقتين (loops) داخل تلك الكتلة (block)، تحتوي كل منهما على استدعاء `trpl::sleep` الذي ينتظر لمدة نصف ثانية (500 مللي ثانية) قبل إرسال الرسالة التالية. نضع حلقة واحدة في جسم `trpl::spawn_task` والأخرى في حلقة `for` ذات مستوى أعلى. نضيف أيضًا `await` بعد استدعاءات `sleep`.

يتصرف هذا الكود بشكل مشابه للتنفيذ القائم على الـ threads - بما في ذلك حقيقة أنك قد ترى الرسائل تظهر بترتيب مختلف في جهازك الطرفي (terminal) عند تشغيله:

```text
hi number 1 from the second task!
hi number 1 from the first task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
```

تتوقف هذه النسخة بمجرد انتهاء حلقة `for` في جسم كتلة async الرئيسية، لأن المهمة (task) التي تم إنشاؤها بواسطة `spawn_task` يتم إغلاقها عند انتهاء دالة `main`. إذا كنت تريد تشغيلها حتى اكتمال المهمة، فستحتاج إلى استخدام مقبض انضمام (join handle) لانتظار اكتمال المهمة الأولى. مع الـ threads، استخدمنا دالة `join` لـ "حظر" (block) الخيط حتى ينتهي من العمل. في القائمة 17-7، يمكننا استخدام `await` لفعل الشيء نفسه، لأن مقبض المهمة (task handle) نفسه هو future. نوع مخرجاته (Output type) هو `Result` لذا نقوم أيضًا بفك تغليفه (unwrap) بعد انتظاره بـ `await`.

<Listing number="17-7" caption="استخدام `await` مع join handle لتشغيل مهمة حتى الاكتمال" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-07/src/main.rs:handle}}
```

</Listing>

هذه النسخة المحدثة تعمل حتى تنتهي _كلتا_ الحلقتين:

```text
hi number 1 from the second task!
hi number 1 from the first task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
hi number 6 from the first task!
hi number 7 from the first task!
hi number 8 from the first task!
hi number 9 from the first task!
```

حتى الآن، يبدو أن async والـ threads يعطياننا نتائج مماثلة، فقط ببناء جملة مختلف: استخدام `await` بدلاً من استدعاء `join` على الـ join handle، وانتظار استدعاءات `sleep` بـ `await`.

الاختلاف الأكبر هو أننا لم نكن بحاجة إلى إنشاء خيط نظام تشغيل (operating system thread) آخر للقيام بذلك. في الواقع، لا نحتاج حتى إلى إنشاء مهمة (task) هنا. نظرًا لأن كتل async تترجم إلى futures مجهولة، يمكننا وضع كل حلقة في كتلة async وجعل وقت التشغيل (runtime) يشغلهما معًا حتى الاكتمال باستخدام دالة `trpl::join`.

في قسم ["انتظار انتهاء جميع الخيوط"][join-handles]<!-- ignore --> في الفصل 16، أظهرنا كيفية استخدام دالة `join` على نوع `JoinHandle` المعاد عند استدعاء `std::thread::spawn`. دالة `trpl::join` مشابهة، ولكن للـ futures. عندما تعطيها اثنين من الـ futures، فإنها تنتج future جديدًا واحدًا يكون مخرجه عبارة عن صف (tuple) يحتوي على مخرجات كل future مررته بمجرد اكتمالهما _معًا_. وبالتالي، في القائمة 17-8، نستخدم `trpl::join` لانتظار انتهاء كل من `fut1` و `fut2`. نحن _لا_ ننتظر `fut1` و `fut2` بـ `await` ولكن بدلاً من ذلك ننتظر الـ future الجديد الناتج عن `trpl::join`. نتجاهل المخرجات لأنها مجرد tuple يحتوي على قيمتين فارغتين (unit values).

<Listing number="17-8" caption="استخدام `trpl::join` لانتظار اثنين من الـ futures المجهولة" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-08/src/main.rs:join}}
```

</Listing>

عندما نشغل هذا، نرى كلا الـ futures يعملان حتى الاكتمال:

```text
hi number 1 from the first task!
hi number 1 from the second task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
hi number 6 from the first task!
hi number 7 from the first task!
hi number 8 from the first task!
hi number 9 from the first task!
```

الآن، سترى نفس الترتيب تمامًا في كل مرة، وهو أمر مختلف تمامًا عما رأيناه مع الـ threads ومع `trpl::spawn_task` في القائمة 17-7. ذلك لأن دالة `trpl::join` هي دالة _عادلة_ (fair)، مما يعني أنها تتحقق من كل future بشكل متساوٍ، بالتناوب بينهما، ولا تسمح أبدًا لأحدهما بالسباق إذا كان الآخر جاهزًا. مع الـ threads، يقرر نظام التشغيل أي خيط يتحقق منه ومدة تركه يعمل. مع async Rust، يقرر الـ runtime أي مهمة يتحقق منها. (في الممارسة العملية، تصبح التفاصيل معقدة لأن الـ async runtime قد يستخدم threads نظام التشغيل داخليًا كجزء من كيفية إدارته للتزامن، لذا فإن ضمان العدالة يمكن أن يكون عملاً إضافيًا للـ runtime - ولكنه لا يزال ممكنًا!) لا يتعين على الـ runtimes ضمان العدالة لأي عملية معينة، وغالبًا ما يقدمون APIs مختلفة لتتيح لك اختيار ما إذا كنت تريد العدالة أم لا.

جرب بعض هذه الاختلافات في انتظار الـ futures وشاهد ما تفعله:

- قم بإزالة كتلة async من حول إحدى الحلقتين أو كلتيهما.
- انتظر كل كتلة async بـ `await` فور تعريفها.
- قم بتغليف الحلقة الأولى فقط في كتلة async، وانتظر الـ future الناتج بعد جسم الحلقة الثانية.

لتحدي إضافي، انظر ما إذا كان بإمكانك معرفة المخرجات في كل حالة _قبل_ تشغيل الكود!

<!-- Old headings. Do not remove or links may break. -->

<a id="message-passing"></a>
<a id="counting-up-on-two-tasks-using-message-passing"></a>

### إرسال البيانات بين مهمتين باستخدام تمرير الرسائل (Sending Data Between Two Tasks Using Message Passing)

ستكون مشاركة البيانات بين الـ futures مألوفة أيضًا: سنستخدم تمرير الرسائل (message passing) مرة أخرى، ولكن هذه المرة مع نسخ async من الأنواع والدوال. سنسلك مسارًا مختلفًا قليلاً عما فعلناه في قسم ["نقل البيانات بين الخيوط باستخدام تمرير الرسائل"][message-passing-threads]<!-- ignore --> في الفصل 16 لتوضيح بعض الاختلافات الرئيسية بين التزامن القائم على الـ threads والقائم على الـ futures. في القائمة 17-9، سنبدأ بكتلة async واحدة فقط - _بدون_ إنشاء مهمة منفصلة كما أنشأنا خيطًا منفصلاً.

<Listing number="17-9" caption="إنشاء قناة async وتعيين النصفين لـ `tx` و `rx`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-09/src/main.rs:channel}}
```

</Listing>

هنا، نستخدم `trpl::channel` وهي نسخة async من API القناة متعددة المنتجين ووحيدة المستهلك (multiple-producer, single-consumer channel) التي استخدمناها مع الـ threads في الفصل 16. نسخة الـ async من الـ API تختلف قليلاً فقط عن النسخة القائمة على الـ threads: فهي تستخدم مستقبل (receiver) `rx` قابل للتغيير (mutable) بدلاً من غير قابل للتغيير، ودالة `recv` الخاصة بها تنتج future نحتاج لانتظاره بـ `await` بدلاً من إنتاج القيمة مباشرة. الآن يمكننا إرسال رسائل من المرسل (sender) إلى المستقبل. لاحظ أننا لا نحتاج لإنشاء thread منفصل أو حتى task؛ نحن نحتاج فقط لانتظار استدعاء `rx.recv` بـ `await`.

دالة `Receiver::recv` المتزامنة في `std::mpsc::channel` تحظر (blocks) حتى تستقبل رسالة. دالة `trpl::Receiver::recv` لا تفعل ذلك، لأنها async. بدلاً من الحظر، فإنها تعيد التحكم إلى الـ runtime حتى يتم استقبال رسالة أو يغلق جانب الإرسال من القناة. في المقابل، نحن لا ننتظر استدعاء `send` بـ `await` لأنه لا يحظر. لا يحتاج لذلك لأن القناة التي نرسل إليها غير محدودة (unbounded).

> ملاحظة: لأن كل كود الـ async هذا يعمل في كتلة async في استدعاء `trpl::block_on` فإن كل شيء داخله يمكنه تجنب الحظر. ومع ذلك، فإن الكود _خارجه_ سيحظر عند انتظار عودة دالة `block_on`. هذا هو الهدف الكامل من دالة `trpl::block_on`: فهي تتيح لك _اختيار_ مكان الحظر على مجموعة من أكواد async، وبالتالي مكان الانتقال بين الكود المتزامن (sync) وغير المتزامن (async).

لاحظ شيئين حول هذا المثال. أولاً، ستصل الرسالة على الفور. ثانياً، على الرغم من أننا نستخدم future هنا، لا يوجد تزامن بعد. كل شيء في القائمة يحدث بالتسلسل، تمامًا كما لو لم تكن هناك futures متضمنة.

دعنا نعالج الجزء الأول عن طريق إرسال سلسلة من الرسائل والنوم بينها، كما هو موضح في القائمة 17-10.

<Listing number="17-10" caption="إرسال واستقبال رسائل متعددة عبر قناة async والنوم مع `await` بين كل رسالة" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-10/src/main.rs:many-messages}}
```

</Listing>

بالإضافة إلى إرسال الرسائل، نحتاج لاستقبالها. في هذه الحالة، لأننا نعرف عدد الرسائل القادمة، يمكننا فعل ذلك يدويًا عن طريق استدعاء `rx.recv().await` أربع مرات. في العالم الحقيقي، سنستخدم عمومًا حلقة معالجة.

في القائمة 16-10، استخدمنا حلقة `for` لمعالجة جميع العناصر المستلمة من قناة متزامنة. ومع ذلك، لا يملك Rust بعد طريقة لاستخدام حلقة `for` مع سلسلة من العناصر _المنتجة بشكل غير متزامن_، لذا نحتاج لاستخدام حلقة لم نرها من قبل: حلقة `while let` الشرطية. هذه هي نسخة الحلقة من بنية `if let` التي رأيناها في قسم ["التحكم في التدفق الموجز باستخدام `if let` و `let...else`"][if-let]<!-- ignore --> في الفصل 6. ستستمر الحلقة في التنفيذ طالما استمر النمط الذي تحدده في مطابقة القيمة.

استدعاء `rx.recv` ينتج future ننتظره بـ `await`. سيقوم الـ runtime بإيقاف الـ future مؤقتًا حتى يصبح جاهزًا. بمجرد وصول رسالة، سيتحول الـ future إلى `Some(message)` لعدد المرات التي تصل فيها رسالة. عندما تغلق القناة، وبغض النظر عما إذا كانت _أي_ رسائل قد وصلت، سيتحول الـ future بدلاً من ذلك إلى `None` للإشارة إلى عدم وجود المزيد من القيم وبالتالي يجب أن نتوقف عن الاستطلاع (polling) - أي نتوقف عن الانتظار بـ `await`.

حلقة `while let` تجمع كل هذا معًا. إذا كانت نتيجة استدعاء `rx.recv().await` هي `Some(message)` فنحن نحصل على الوصول للرسالة ويمكننا استخدامها في جسم الحلقة، تمامًا كما نفعل مع `if let`. إذا كانت النتيجة `None` تنتهي الحلقة. في كل مرة تكتمل فيها الحلقة، تصل لنقطة الـ await مرة أخرى، لذا يقوم الـ runtime بإيقافها مؤقتًا مرة أخرى حتى تصل رسالة أخرى.

الكود الآن يرسل ويستقبل جميع الرسائل بنجاح. لسوء الحظ، لا تزال هناك بضع مشاكل. أولاً، الرسائل لا تصل بفواصل زمنية مدتها نصف ثانية. إنها تصل جميعها دفعة واحدة، بعد ثانيتين (2000 مللي ثانية) من بدء البرنامج. ثانياً، هذا البرنامج لا ينتهي أبداً! بدلاً من ذلك، ينتظر للأبد رسائل جديدة. ستحتاج لإغلاقه باستخدام <kbd>ctrl</kbd>-<kbd>C</kbd>.

#### الكود داخل كتلة Async واحدة ينفذ خطياً (Code Within One Async Block Executes Linearly)

لنبدأ بفحص سبب وصول الرسائل دفعة واحدة بعد التأخير الكامل، بدلاً من وصولها مع تأخيرات بين كل واحدة. داخل كتلة async معينة، يكون الترتيب الذي تظهر به الكلمات المفتاحية `await` في الكود هو أيضاً الترتيب الذي يتم تنفيذها به عند تشغيل البرنامج.

توجد كتلة async واحدة فقط في القائمة 17-10، لذا كل شيء فيها يعمل خطياً. لا يزال لا يوجد تزامن. تحدث جميع استدعاءات `tx.send` متخللة بجميع استدعاءات `trpl::sleep` ونقاط الـ await المرتبطة بها. عندها فقط تصل حلقة `while let` لأي من نقاط الـ `await` في استدعاءات `recv`.

للحصول على السلوك الذي نريده، حيث يحدث تأخير النوم بين كل رسالة، نحتاج لوضع عمليات `tx` و `rx` في كتل async الخاصة بها، كما هو موضح في القائمة 17-11. عندها يمكن للـ runtime تنفيذ كل منها بشكل منفصل باستخدام `trpl::join` تمامًا كما في القائمة 17-8. مرة أخرى، ننتظر نتيجة استدعاء `trpl::join` بـ `await` وليس الـ futures الفردية. إذا انتظرنا الـ futures الفردية بالتسلسل، فسينتهي بنا الأمر مرة أخرى في تدفق تسلسلي - وهو بالضبط ما نحاول _عدم_ فعله.

<Listing number="17-11" caption="فصل `send` و `recv` في كتل `async` الخاصة بهما وانتظار الـ futures لتلك الكتل" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-11/src/main.rs:futures}}
```

</Listing>

مع الكود المحدث في القائمة 17-11، يتم طباعة الرسائل بفواصل زمنية مدتها 500 مللي ثانية، بدلاً من وصولها جميعاً دفعة واحدة بعد ثانيتين.

#### نقل الملكية إلى كتلة Async (Moving Ownership Into an Async Block)

البرنامج لا يزال لا ينتهي أبداً، بسبب الطريقة التي تتفاعل بها حلقة `while let` مع `trpl::join`:

- الـ future المعاد من `trpl::join` يكتمل فقط بمجرد اكتمال _كلا_ الـ futures الممررة إليه.
- الـ future المسمى `tx_fut` يكتمل بمجرد انتهائه من النوم بعد إرسال آخر رسالة في `vals`.
- الـ future المسمى `rx_fut` لن يكتمل حتى تنتهي حلقة `while let`.
- حلقة `while let` لن تنتهي حتى ينتج انتظار `rx.recv` القيمة `None`.
- انتظار `rx.recv` سيعيد `None` فقط بمجرد إغلاق الطرف الآخر من القناة.
- ستغلق القناة فقط إذا استدعينا `rx.close` أو عندما يتم إسقاط (drop) جانب المرسل `tx`.
- نحن لا نستدعي `rx.close` في أي مكان، ولن يتم إسقاط `tx` حتى تنتهي كتلة async الخارجية الممررة لـ `trpl::block_on`.
- لا يمكن للكتلة أن تنتهي لأنها محظورة بانتظار اكتمال `trpl::join` مما يعيدنا لأعلى هذه القائمة.

في الوقت الحالي، كتلة async حيث نرسل الرسائل تقوم فقط بـ _اقتراض_ `tx` لأن إرسال رسالة لا يتطلب ملكية (ownership)، ولكن إذا استطعنا _نقل_ (move) `tx` إلى كتلة async تلك، فسيتم إسقاطها بمجرد انتهاء تلك الكتلة. في قسم ["التقاط المراجع أو نقل الملكية"][capture-or-move]<!-- ignore --> في الفصل 13، تعلمت كيفية استخدام الكلمة المفتاحية `move` مع الـ closures، وكما نوقش في قسم ["استخدام `move` closures مع الـ Threads"][move-threads]<!-- ignore --> في الفصل 16، غالباً ما نحتاج لنقل البيانات إلى الـ closures عند العمل مع الـ threads. تنطبق نفس الديناميكيات الأساسية على كتل async، لذا تعمل الكلمة المفتاحية `move` مع كتل async تماماً كما تعمل مع الـ closures.

في القائمة 17-12، نقوم بتغيير الكتلة المستخدمة لإرسال الرسائل من `async` إلى `async move`.

<Listing number="17-12" caption="مراجعة للكود من القائمة 17-11 تغلق البرنامج بشكل صحيح عند الاكتمال" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-12/src/main.rs:with-move}}
```

</Listing>

عندما نشغل _هذه_ النسخة من الكود، فإنها تغلق برشاقة بعد إرسال واستقبال آخر رسالة. بعد ذلك، لنرى ما الذي سيحتاج للتغيير لإرسال البيانات من أكثر من future واحد.

#### ربط عدد من الـ Futures باستخدام ماكرو `join!` (Joining a Number of Futures with the `join!` Macro)

قناة الـ async هذه هي أيضاً قناة متعددة المنتجين، لذا يمكننا استدعاء `clone` على `tx` إذا أردنا إرسال رسائل من عدة futures، كما هو موضح في القائمة 17-13.

<Listing number="17-13" caption="استخدام منتجين متعددين مع كتل async" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-13/src/main.rs:here}}
```

</Listing>

أولاً، نقوم باستنساخ (clone) لـ `tx` لننشئ `tx1` خارج كتلة async الأولى. ننقل `tx1` إلى تلك الكتلة تماماً كما فعلنا سابقاً مع `tx`. ثم لاحقاً، ننقل `tx` الأصلي إلى كتلة async _جديدة_، حيث نرسل المزيد من الرسائل بتأخير أبطأ قليلاً. لقد وضعنا كتلة async الجديدة هذه بعد كتلة async الخاصة باستقبال الرسائل، ولكن كان يمكن وضعها قبلها أيضاً. المفتاح هو الترتيب الذي يتم به انتظار الـ futures بـ `await` وليس الترتيب الذي تم إنشاؤها به.

كلتا كتلتي async لإرسال الرسائل تحتاجان لتكونا كتل `async move` بحيث يتم إسقاط كل من `tx` و `tx1` عند انتهاء تلك الكتل. وإلا، فسينتهي بنا الأمر مرة أخرى في نفس الحلقة اللانهائية التي بدأنا بها.

أخيراً، ننتقل من `trpl::join` إلى `trpl::join!` للتعامل مع الـ future الإضافي: ماكرو `join!` ينتظر عدداً عشوائياً من الـ futures بـ `await` حيث نعرف عدد الـ futures في وقت الترجمة (compile time). سنناقش انتظار مجموعة من عدد غير معروف من الـ futures لاحقاً في هذا الفصل.

الآن نرى جميع الرسائل من كلا الـ futures المرسلين، ولأن الـ futures المرسلين يستخدمون تأخيرات مختلفة قليلاً بعد الإرسال، يتم استقبال الرسائل أيضاً بتلك الفواصل الزمنية المختلفة:

```text
received 'hi'
received 'more'
received 'from'
received 'the'
received 'messages'
received 'future'
received 'for'
received 'you'
```

لقد استكشفنا كيفية استخدام تمرير الرسائل لإرسال البيانات بين الـ futures، وكيف يعمل الكود داخل كتلة async بشكل تسلسلي، وكيفية نقل الملكية إلى كتلة async، وكيفية ربط عدة futures. بعد ذلك، دعنا نناقش كيف ولماذا نخبر الـ runtime أنه يمكنه الانتقال لمهمة أخرى.

[thread-spawn]: ch16-01-threads.html#creating-a-new-thread-with-spawn
[join-handles]: ch16-01-threads.html#waiting-for-all-threads-to-finish
[message-passing-threads]: ch16-02-message-passing.html
[if-let]: ch06-03-if-let.html
[capture-or-move]: ch13-01-closures.html#capturing-references-or-moving-ownership
[move-threads]: ch16-01-threads.html#using-move-closures-with-threads
