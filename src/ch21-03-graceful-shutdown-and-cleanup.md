## الإغلاق الآمن (Graceful Shutdown) والتنظيف (Cleanup)

يستجيب الكود في القائمة 21-20 للطلبات بشكل غير متزامن (asynchronously) من خلال استخدام مجمع خيوط المعالجة (thread pool)، كما خططنا. تظهر لنا بعض التحذيرات حول حقول (fields) `workers` و `id` و `thread` التي لا نستخدمها بطريقة مباشرة، مما يذكرنا بأننا لا نقوم بأي عملية تنظيف. عندما نستخدم الطريقة الأقل أناقة بالضغط على <kbd>ctrl</kbd>-<kbd>C</kbd> لإيقاف خيط المعالجة الرئيسي (main thread)، يتم إيقاف جميع خيوط المعالجة الأخرى فوراً أيضاً، حتى لو كانت في منتصف خدمة طلب ما.

بعد ذلك، سنقوم بتنفيذ سمة (trait) `Drop` لاستدعاء `join` على كل خيط من خيوط المعالجة في pool للتأكد من إنهاء الطلبات التي تعمل عليها قبل الإغلاق. ثم سنقوم بتنفيذ طريقة لإخبار خيوط المعالجة بوجوب التوقف عن قبول طلبات جديدة والإغلاق. لرؤية هذا الكود قيد التشغيل، سنقوم بتعديل الخادم الخاص بنا ليقبل طلبين فقط قبل إغلاق thread pool الخاص به بشكل آمن.

شيء واحد يجب ملاحظته أثناء المضي قدماً: لا يؤثر أي من هذا على أجزاء الكود التي تتعامل مع تنفيذ الإغلاقات (closures)، لذا سيكون كل شيء هنا هو نفسه إذا كنا نستخدم thread pool لـ وقت تشغيل غير متزامن (async runtime).

### تنفيذ سمة `Drop` على `ThreadPool`

لنبدأ بتنفيذ `Drop` على thread pool الخاص بنا. عندما يتم حذف (dropped) pool، يجب أن تنضم (join) جميع خيوط المعالجة الخاصة بنا للتأكد من إنهاء عملها. تعرض القائمة 21-22 محاولة أولى لتنفيذ `Drop` ؛ هذا الكود لن يعمل تماماً بعد.

<Listing number="21-22" file-name="src/lib.rs" caption="القيام بـ join لكل خيط عندما يخرج thread pool عن النطاق">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/listing-21-22/src/lib.rs:here}}
```

</Listing>

أولاً، نقوم بالمرور عبر كل من `workers` في thread pool. نستخدم `&mut` لهذا لأن `self` هو مرجع قابل للتغيير، ونحتاج أيضاً إلى القدرة على تغيير `worker`. لكل `worker` ، نقوم بطباعة رسالة تفيد بأن مثيل (instance) `Worker` هذا قيد الإغلاق، ثم نستدعي `join` على خيط ذلك instance. إذا فشل استدعاء `join` ، نستخدم `unwrap` لجعل Rust تدخل في حالة هلع (panic) وتنتقل إلى إغلاق غير آمن.

إليك الخطأ الذي نحصل عليه عند تصريف (compile) هذا الكود:

```console
{{#include ../listings/ch21-web-server/listing-21-22/output.txt}}
```

يخبرنا الخطأ أنه لا يمكننا استدعاء `join` لأننا نملك فقط استعارة قابلة للتغيير (mutable borrow) لكل `worker` ، بينما تأخذ `join` ملكية (ownership) وسيطها. لحل هذه المشكلة، نحتاج إلى نقل الخيط خارج instance `Worker` الذي يملك `thread` حتى تتمكن `join` من استهلاك الخيط. إحدى الطرق للقيام بذلك هي اتباع نفس النهج الذي اتبعناه في القائمة 18-15. إذا كان `Worker` يحتفظ بـ `Option<thread::JoinHandle<()>>` ، فيمكننا استدعاء دالة `take` على `Option` لنقل القيمة خارج متغير (variant) `Some` وترك variant `None` في مكانه. بعبارة أخرى، فإن `Worker` الذي يعمل سيكون لديه variant `Some` في `thread` ، وعندما نريد تنظيف `Worker` ، سنستبدل `Some` بـ `None` حتى لا يكون لدى `Worker` خيط لتشغيله.

ومع ذلك، فإن المرة _الوحيدة_ التي سيظهر فيها هذا هي عند حذف `Worker`. في المقابل، سيتعين علينا التعامل مع `Option<thread::JoinHandle<()>>` في أي مكان نصل فيه إلى `worker.thread`. يستخدم أسلوب رست الاصطلاحي (Idiomatic Rust) نوع `Option` كثيراً، ولكن عندما تجد نفسك تغلف شيئاً تعرف أنه سيكون موجوداً دائماً في `Option` كحل بديل مثل هذا، فمن الجيد البحث عن طرق بديلة لجعل الكود الخاص بك أنظف وأقل عرضة للخطأ.

في هذه الحالة، يوجد بديل أفضل: دالة `Vec::drain`. وهي تقبل معلمة نطاق (range parameter) لتحديد العناصر التي سيتم إزالتها من المتجه (vector) وتُرجع مكرراً (iterator) لتلك العناصر. سيؤدي تمرير بناء جملة النطاق `..` إلى إزالة *كل* قيمة من vector.

لذا، نحتاج إلى تحديث تنفيذ `drop` لـ `ThreadPool` كالتالي:

<Listing file-name="src/lib.rs">

```rust
{{#rustdoc_include ../listings/ch21-web-server/no-listing-04-update-drop-definition/src/lib.rs:here}}
```

</Listing>

يحل هذا خطأ compiler ولا يتطلب أي تغييرات أخرى في الكود الخاص بنا. لاحظ أنه نظراً لأنه يمكن استدعاء drop عند حدوث panic، فإن unwrap يمكن أن تسبب أيضاً panic وتؤدي إلى هلع مزدوج (double panic)، مما يؤدي فوراً إلى تعطل البرنامج وإنهاء أي عملية تنظيف قيد التنفيذ. هذا أمر مقبول لبرنامج تجريبي، ولكنه غير مستحسن لكود الإنتاج.

### إرسال إشارة إلى خيوط المعالجة للتوقف عن انتظار المهام

مع كل التغييرات التي أجريناها، يتم تصريف الكود الخاص بنا دون أي تحذيرات. ومع ذلك، فإن الخبر السيئ هو أن هذا الكود لا يعمل بالطريقة التي نريدها بعد. المفتاح هو المنطق في closures التي يتم تشغيلها بواسطة خيوط المعالجة لـ instances `Worker`: في الوقت الحالي، نستدعي `join` ، لكن ذلك لن يوقف خيوط المعالجة، لأنها في حلقة (loop) للأبد تبحث عن مهام (jobs). إذا حاولنا حذف `ThreadPool` مع تنفيذنا الحالي لـ `drop` ، فسيتم حظر (block) main thread للأبد، بانتظار انتهاء الخيط الأول.

لإصلاح هذه المشكلة، سنحتاج إلى تغيير في تنفيذ `drop` لـ `ThreadPool` ثم تغيير في loop الخاص بـ `Worker`.

أولاً، سنقوم بتغيير تنفيذ `drop` لـ `ThreadPool` لحذف المرسل (sender) صراحةً قبل انتظار انتهاء خيوط المعالجة. تعرض القائمة 21-23 التغييرات على `ThreadPool` لحذف `sender` صراحةً. على عكس الخيط، نحتاج هنا _بالفعل_ إلى استخدام `Option` لنتمكن من نقل `sender` خارج `ThreadPool` باستخدام `Option::take`.

<Listing number="21-23" file-name="src/lib.rs" caption="حذف sender صراحةً قبل القيام بـ join لخيوط Worker">

```rust,noplayground,not_desired_behavior
{{#rustdoc_include ../listings/ch21-web-server/listing-21-23/src/lib.rs:here}}
```

</Listing>

يؤدي حذف `sender` إلى إغلاق القناة (channel)، مما يشير إلى أنه لن يتم إرسال المزيد من الرسائل. عندما يحدث ذلك، فإن جميع استدعاءات `recv` التي تقوم بها instances `Worker` في الحلقة اللانهائية ستُرجع خطأً. في القائمة 21-24، نقوم بتغيير loop الخاص بـ `Worker` للخروج من الحلقة بشكل آمن في هذه الحالة، مما يعني أن خيوط المعالجة ستنتهي عندما يستدعي تنفيذ `drop` لـ `ThreadPool` دالة `join` عليها.

<Listing number="21-24" file-name="src/lib.rs" caption="الخروج صراحةً من الحلقة عندما تُرجع recv خطأً">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-24/src/lib.rs:here}}
```

</Listing>

لرؤية هذا الكود قيد التشغيل، دعونا نعدل `main` لقبول طلبين فقط قبل إغلاق الخادم بشكل آمن، كما هو موضح في القائمة 21-25.

<Listing number="21-25" file-name="src/main.rs" caption="إغلاق الخادم بعد خدمة طلبين عن طريق الخروج من الحلقة">

```rust,ignore
{{#rustdoc_include ../listings/ch21-web-server/listing-21-25/src/main.rs:here}}
```

</Listing>

لن ترغب في إغلاق خادم ويب حقيقي بعد خدمة طلبين فقط. يوضح هذا الكود فقط أن Graceful Shutdown و Cleanup يعملان بشكل صحيح.

دالة `take` معرفة في trait `Iterator` وتحد من التكرار إلى أول عنصرين على الأكثر. سيخرج `ThreadPool` عن النطاق في نهاية `main` ، وسيتم تشغيل تنفيذ `drop`.

ابدأ الخادم باستخدام `cargo run` وقم بإجراء ثلاثة طلبات. يجب أن يفشل الطلب الثالث، وفي terminal الخاص بك، يجب أن ترى مخرجات مشابهة لهذا:

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.41s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Shutting down.
Shutting down worker 0
Worker 3 got a job; executing.
Worker 1 disconnected; shutting down.
Worker 2 disconnected; shutting down.
Worker 3 disconnected; shutting down.
Worker 0 disconnected; shutting down.
Shutting down worker 1
Shutting down worker 2
Shutting down worker 3
```

قد ترى ترتيباً مختلفاً لمعرفات `Worker` والرسائل المطبوعة. يمكننا أن نرى كيف يعمل هذا الكود من الرسائل: حصلت instances `Worker` رقم 0 و 3 على أول طلبين. توقف الخادم عن قبول الاتصالات بعد الاتصال الثاني، ويبدأ تنفيذ `Drop` على `ThreadPool` قبل أن يبدأ `Worker 3` مهمته. يؤدي حذف `sender` إلى فصل جميع instances `Worker` وإخبارها بالإغلاق. تطبع instances `Worker` رسالة عند فصلها، ثم يستدعي thread pool دالة `join` لانتظار انتهاء كل خيط `Worker`.

لاحظ جانباً مثيراً للاهتمام في هذا التنفيذ المحدد: قام `ThreadPool` بحذف `sender` ، وقبل أن يتلقى أي `Worker` خطأً، حاولنا الانضمام إلى `Worker 0`. لم يكن `Worker 0` قد تلقى خطأً بعد من `recv` ، لذا تم حظر main thread، بانتظار انتهاء `Worker 0`. في هذه الأثناء، تلقى `Worker 3` مهمة ثم تلقت جميع خيوط المعالجة خطأً. عندما انتهى `Worker 0` ، انتظر main thread انتهاء بقية instances `Worker`. عند تلك النقطة، كانت جميعها قد خرجت من حلقاتها وتوقفت.

تهانينا! لقد أكملنا مشروعنا الآن؛ لدينا خادم ويب أساسي يستخدم thread pool للاستجابة بشكل غير متزامن. نحن قادرون على إجراء Graceful Shutdown للخادم، مما ينظف جميع خيوط المعالجة في pool.

إليك الكود الكامل للمرجع:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch21-web-server/no-listing-07-final-code/src/main.rs}}
```

</Listing>

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-07-final-code/src/lib.rs}}
```

</Listing>

يمكننا القيام بالمزيد هنا! إذا كنت ترغب في الاستمرار في تحسين هذا المشروع، فإليك بعض الأفكار:

- إضافة المزيد من التوثيق (documentation) لـ `ThreadPool` ودوالها العامة.
- إضافة اختبارات (tests) لوظائف المكتبة.
- تغيير استدعاءات `unwrap` إلى معالجة أخطاء أكثر قوة.
- استخدام `ThreadPool` لأداء مهام أخرى غير خدمة طلبات الويب.
- العثور على حزمة (crate) لـ thread pool على [crates.io](https://crates.io/) وتنفيذ خادم ويب مماثل باستخدام crate بدلاً من ذلك. ثم قارن واجهة برمجة التطبيقات (API) الخاصة بها وقوتها بـ thread pool الذي قمنا بتنفيذه.

## ملخص (Summary)

أحسنت! لقد وصلت إلى نهاية الكتاب! نود أن نشكرك على انضمامك إلينا في هذه الجولة في Rust. أنت الآن جاهز لتنفيذ مشاريع Rust الخاصة بك والمساعدة في مشاريع الآخرين. ضع في اعتبارك أن هناك مجتمعاً مضيافاً من الـ Rustaceans الآخرين الذين يسعدهم مساعدتك في أي تحديات تواجهها في رحلتك مع Rust.
