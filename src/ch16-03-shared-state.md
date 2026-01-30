## تزامن الحالة المشتركة (Shared-State Concurrency)

يُعد تمرير الرسائل (Message passing) طريقة جيدة للتعامل مع التزامن (concurrency)، ولكنه ليس الطريقة الوحيدة. هناك طريقة أخرى تتمثل في وصول خيوط (threads) متعددة إلى نفس البيانات المشتركة (shared data). تذكر هذا الجزء من شعار وثائق لغة Go مرة أخرى: "لا تتواصل عن طريق مشاركة الذاكرة (sharing memory)."

كيف سيبدو التواصل عن طريق memory sharing؟ بالإضافة إلى ذلك، لماذا يحذر المتحمسون لـ message-passing من عدم استخدام memory sharing؟

بطريقة ما، تشبه القنوات (channels) في أي لغة برمجة الـ ملكية فردية (single ownership) لأنه بمجرد نقل قيمة عبر channel، لا يجب عليك استخدام تلك القيمة بعد الآن. يشبه تزامن الذاكرة المشتركة (Shared-memory concurrency) الـ ملكية متعددة (multiple ownership): يمكن لـ threads متعددة الوصول إلى نفس موقع الذاكرة في نفس الوقت. كما رأيت في الفصل 15، حيث جعلت الـ مؤشرات الذكية (smart pointers) الـ multiple ownership ممكنة، يمكن أن تضيف الـ multiple ownership تعقيدًا لأن هؤلاء المالكين المختلفين يحتاجون إلى إدارة. يساعد نظام الأنواع (type system) وقواعد الـ ownership في Rust بشكل كبير في جعل هذه الإدارة صحيحة. كمثال، دعنا ننظر إلى الـ mutexes، وهي إحدى بدائيات التزامن (concurrency primitives) الأكثر شيوعًا للـ shared memory.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-mutexes-to-allow-access-to-data-from-one-thread-at-a-time"></a>

### التحكم في الوصول باستخدام المزاليج (Mutexes)

الـ **Mutex** هو اختصار لـ *الاستبعاد المتبادل* (mutual exclusion)، بمعنى أن الـ mutex يسمح لـ thread واحد فقط بالوصول إلى بعض الـ data في أي وقت معين. للوصول إلى الـ data في mutex، يجب على الـ thread أولاً الإشارة إلى أنه يريد الوصول عن طريق طلب الحصول على قفل (lock) الـ mutex. الـ lock هو بنية بيانات (data structure) تعد جزءًا من الـ mutex وتتتبع من لديه حاليًا وصول حصري إلى الـ data. لذلك، يوصف الـ mutex بأنه *يحمي* (guarding) الـ data التي يحملها عبر نظام الـ locking.

تشتهر الـ mutexes بصعوبة استخدامها لأنه يجب عليك تذكر قاعدتين:

1. يجب أن تحاول الحصول على الـ lock قبل استخدام الـ data.
2. عندما تنتهي من الـ data التي يحميها الـ mutex، يجب عليك إلغاء قفل (unlock) الـ data حتى تتمكن الـ threads الأخرى من الحصول على الـ lock.

بالنسبة لاستعارة من العالم الحقيقي لـ mutex، تخيل حلقة نقاش في مؤتمر بها ميكروفون واحد فقط. قبل أن يتمكن أحد أعضاء اللجنة من التحدث، يجب عليه أن يطلب أو يشير إلى أنه يريد استخدام الميكروفون. عندما يحصل على الميكروفون، يمكنه التحدث للمدة التي يريدها ثم يسلم الميكروفون إلى عضو اللجنة التالي الذي يطلب التحدث. إذا نسي أحد أعضاء اللجنة تسليم الميكروفون عند الانتهاء منه، فلن يتمكن أي شخص آخر من التحدث. إذا ساءت إدارة الميكروفون المشترك، فلن تعمل حلقة النقاش كما هو مخطط لها!

قد تكون إدارة الـ mutexes صعبة للغاية، ولهذا السبب فإن الكثير من الناس متحمسون لـ channels. ومع ذلك، بفضل الـ type system وقواعد الـ ownership في Rust، لا يمكنك أن تخطئ في الـ locking والـ unlocking.

#### واجهة برمجة التطبيقات لـ `Mutex<T>`

كمثال على كيفية استخدام mutex، دعنا نبدأ باستخدام mutex في سياق thread واحد، كما هو موضح في القائمة 16-12.

<Listing number="16-12" file-name="src/main.rs" caption="استكشاف واجهة برمجة التطبيقات لـ `Mutex<T>` في سياق thread واحد للتبسيط">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-12/src/main.rs}}
```

</Listing>

كما هو الحال مع العديد من الـ types، ننشئ `Mutex<T>` باستخدام الدالة المرتبطة (associated function) `new`. للوصول إلى الـ data داخل الـ mutex، نستخدم method `lock` للحصول على الـ lock. سيؤدي هذا الاستدعاء إلى حظر الـ thread الحالي بحيث لا يمكنه القيام بأي عمل حتى يحين دورنا للحصول على الـ lock.

سيفشل استدعاء `lock` إذا أصيب thread آخر يحمل الـ lock بالذعر (panicked). في هذه الحالة، لن يتمكن أحد من الحصول على الـ lock، لذلك اخترنا `unwrap` وجعل هذا الـ thread يصاب بالذعر إذا كنا في هذا الموقف.

بعد أن حصلنا على الـ lock، يمكننا التعامل مع القيمة المرجعة، المسماة `num` في هذه الحالة، كـ mutable reference للـ data الداخلية. يضمن الـ type system أننا نحصل على lock قبل استخدام القيمة في `m`. نوع `m` هو `Mutex<i32>`، وليس `i32`، لذلك *يجب* علينا استدعاء `lock` لنتمكن من استخدام قيمة `i32`. لا يمكننا أن ننسى؛ لن يسمح لنا الـ type system بالوصول إلى الـ `i32` الداخلي بخلاف ذلك.

يعيد استدعاء `lock` نوعًا يسمى **MutexGuard**، ملفوفًا في `LockResult` الذي تعاملنا معه باستدعاء `unwrap`. يطبق نوع `MutexGuard` سمة `Deref` للإشارة إلى الـ data الداخلية الخاصة بنا؛ يحتوي الـ type أيضًا على تطبيق الإسقاط (Drop implementation) الذي يحرر الـ lock تلقائيًا عندما يخرج `MutexGuard` من النطاق (scope)، وهو ما يحدث في نهاية الـ scope الداخلي. نتيجة لذلك، لا نخاطر بنسيان تحرير الـ lock وحظر الـ mutex من استخدامه بواسطة threads أخرى لأن تحرير الـ lock يحدث تلقائيًا.

بعد إسقاط الـ lock، يمكننا طباعة قيمة الـ mutex ونرى أننا تمكنا من تغيير الـ `i32` الداخلي إلى `6`.

<!-- Old headings. Do not remove or links may break. -->

<a id="sharing-a-mutext-between-multiple-threads"></a>

#### الوصول المشترك إلى `Mutex<T>`

الآن دعنا نحاول مشاركة قيمة بين threads متعددة باستخدام `Mutex<T>`. سنقوم بتشغيل 10 threads ونجعل كل واحد منها يزيد قيمة عداد (counter) بمقدار 1، بحيث ينتقل الـ counter من 0 إلى 10. سيحتوي المثال في القائمة 16-13 على خطأ compiler، وسنستخدم هذا الخطأ لمعرفة المزيد حول استخدام `Mutex<T>` وكيف تساعدنا Rust في استخدامه بشكل صحيح.

<Listing number="16-13" file-name="src/main.rs" caption="عشرة threads، كل واحد يزيد عدادًا محميًا بواسطة `Mutex<T>`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-13/src/main.rs}}
```

</Listing>

ننشئ متغير `counter` لاحتواء `i32` داخل `Mutex<T>`، كما فعلنا في القائمة 16-12. بعد ذلك، ننشئ 10 threads عن طريق التكرار على نطاق من الأرقام. نستخدم `thread::spawn` ونعطي جميع الـ threads نفس الـ closure: واحد ينقل الـ counter إلى الـ thread، ويحصل على lock على `Mutex<T>` عن طريق استدعاء method `lock`، ثم يضيف 1 إلى القيمة في الـ mutex. عندما ينتهي الـ thread من تشغيل الـ closure الخاص به، سيخرج `num` من الـ scope ويحرر الـ lock حتى يتمكن thread آخر من الحصول عليه.

في الـ main thread، نجمع جميع مقابض الانضمام (join handles). بعد ذلك، كما فعلنا في القائمة 16-2، نستدعي `join` على كل handle للتأكد من انتهاء جميع الـ threads. عند هذه النقطة، سيحصل الـ main thread على الـ lock ويطبع نتيجة هذا البرنامج.

لقد أشرنا إلى أن هذا المثال لن يتم تجميعه. الآن دعنا نكتشف السبب!

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-13/output.txt}}
```

تنص رسالة الخطأ على أن قيمة `counter` قد تم نقلها (moved) في التكرار السابق للـ loop. تخبرنا Rust أنه لا يمكننا نقل الـ ownership لـ lock `counter` إلى threads متعددة. دعنا نصلح خطأ الـ compiler باستخدام طريقة الـ multiple-ownership التي ناقشناها في الفصل 15.

#### الـ Multiple Ownership مع الـ Threads المتعددة

في الفصل 15، أعطينا قيمة لـ owners متعددين باستخدام الـ smart pointer `Rc<T>` لإنشاء قيمة مرجعية العد (reference-counted). دعنا نفعل الشيء نفسه هنا ونرى ما سيحدث. سنقوم بلف `Mutex<T>` في `Rc<T>` في القائمة 16-14 واستنساخ (clone) الـ `Rc<T>` قبل نقل الـ ownership إلى الـ thread.

<Listing number="16-14" file-name="src/main.rs" caption="محاولة استخدام `Rc<T>` للسماح لـ threads متعددة بامتلاك `Mutex<T>`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-14/src/main.rs}}
```

</Listing>

مرة أخرى، نقوم بالـ compile ونحصل على... أخطاء مختلفة! الـ compiler يعلمنا الكثير:

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-14/output.txt}}
```

يا إلهي، رسالة الخطأ مطولة جدًا! إليك الجزء المهم الذي يجب التركيز عليه: `` `Rc<Mutex<i32>>` cannot be sent between threads safely ``. يخبرنا الـ compiler أيضًا بالسبب: `` the trait `Send` is not implemented for `Rc<Mutex<i32>>` ``. سنتحدث عن `Send` في القسم التالي: إنها إحدى الـ traits التي تضمن أن الـ types التي نستخدمها مع الـ threads مخصصة للاستخدام في مواقف التزامن (concurrent situations).

لسوء الحظ، `Rc<T>` ليس آمنًا للمشاركة عبر الـ threads. عندما يدير `Rc<T>` عداد الـ reference، فإنه يضيف إلى العداد لكل استدعاء لـ `clone` ويطرح من العداد عندما يتم إسقاط كل clone. لكنه لا يستخدم أي concurrency primitives للتأكد من أن التغييرات على العداد لا يمكن أن يقاطعها thread آخر. قد يؤدي هذا إلى أعداد خاطئة - أخطاء خفية يمكن أن تؤدي بدورها إلى تسرب الذاكرة (memory leaks) أو إسقاط قيمة قبل الانتهاء منها. ما نحتاجه هو نوع يشبه تمامًا `Rc<T>`، ولكنه يجري تغييرات على عداد الـ reference بطريقة آمنة للـ thread.

#### عد مرجعي ذري (Atomic Reference Counting) باستخدام `Arc<T>`

لحسن الحظ، `Arc<T>` *هو* نوع مثل `Rc<T>` آمن للاستخدام في مواقف التزامن. يشير الحرف *a* إلى *ذري* (atomic)، مما يعني أنه نوع *مرجعي العد ذريًا* (atomically reference-counted). الـ Atomics هي نوع إضافي من concurrency primitive لن نغطيه بالتفصيل هنا: راجع وثائق الـ standard library لـ [`std::sync::atomic`][atomic] لمزيد من التفاصيل. في هذه المرحلة، تحتاج فقط إلى معرفة أن الـ atomics تعمل مثل الـ primitive types ولكنها آمنة للمشاركة عبر الـ threads.

قد تتساءل إذن لماذا ليست جميع الـ primitive types ذرية ولماذا لم يتم تطبيق الـ standard library types لاستخدام `Arc<T>` افتراضيًا. السبب هو أن أمان الـ thread يأتي مع عقوبة في الأداء لا تريد دفعها إلا عندما تحتاج إليها حقًا. إذا كنت تجري عمليات على قيم داخل thread واحد فقط، يمكن أن يعمل الكود الخاص بك بشكل أسرع إذا لم يكن مضطرًا لفرض الضمانات التي توفرها الـ atomics.

دعنا نعود إلى مثالنا: يحتوي `Arc<T>` و `Rc<T>` على نفس واجهة برمجة التطبيقات (API)، لذلك نصلح برنامجنا عن طريق تغيير سطر `use`، واستدعاء `new`، واستدعاء `clone`. سيتم تجميع وتشغيل الكود في القائمة 16-15 أخيرًا.

<Listing number="16-15" file-name="src/main.rs" caption="استخدام `Arc<T>` لتغليف `Mutex<T>` للتمكن من مشاركة الـ ownership عبر threads متعددة">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-15/src/main.rs}}
```

</Listing>

سيقوم هذا الكود بطباعة ما يلي:

```text
Result: 10
```

لقد فعلناها! لقد قمنا بالعد من 0 إلى 10، وهو ما قد لا يبدو مثيرًا للإعجاب، ولكنه علمنا الكثير عن `Mutex<T>` وأمان الـ thread. يمكنك أيضًا استخدام بنية هذا البرنامج لإجراء عمليات أكثر تعقيدًا من مجرد زيادة عداد. باستخدام هذه الاستراتيجية، يمكنك تقسيم عملية حسابية إلى أجزاء مستقلة، وتقسيم هذه الأجزاء عبر الـ threads، ثم استخدام `Mutex<T>` لجعل كل thread يقوم بتحديث النتيجة النهائية بجزئه.

لاحظ أنه إذا كنت تجري عمليات رقمية بسيطة، فهناك أنواع أبسط من أنواع `Mutex<T>` التي توفرها وحدة [`std::sync::atomic` في الـ standard library][atomic]. توفر هذه الـ types وصولًا آمنًا ومتزامنًا وذريًا إلى الـ primitive types. اخترنا استخدام `Mutex<T>` مع primitive type لهذا المثال حتى نتمكن من التركيز على كيفية عمل `Mutex<T>`.

<!-- Old headings. Do not remove or links may break. -->

<a id="similarities-between-refcelltrct-and-mutextarct"></a>

### مقارنة `RefCell<T>`/`Rc<T>` و `Mutex<T>`/`Arc<T>`

ربما لاحظت أن `counter` غير قابل للتغيير (immutable) ولكن يمكننا الحصول على mutable reference للقيمة بداخله؛ هذا يعني أن `Mutex<T>` يوفر قابلية التغيير الداخلية (interior mutability)، كما تفعل عائلة `Cell`. بنفس الطريقة التي استخدمنا بها `RefCell<T>` في الفصل 15 للسماح لنا بتغيير المحتويات داخل `Rc<T>`، نستخدم `Mutex<T>` لتغيير المحتويات داخل `Arc<T>`.

هناك تفصيل آخر يجب ملاحظته وهو أن Rust لا يمكنها حمايتك من جميع أنواع أخطاء المنطق (logic errors) عند استخدام `Mutex<T>`. تذكر من الفصل 15 أن استخدام `Rc<T>` جاء مع خطر إنشاء دورات مرجعية (reference cycles)، حيث يشير قيمتا `Rc<T>` إلى بعضهما البعض، مما يتسبب في memory leaks. وبالمثل، يأتي `Mutex<T>` مع خطر إنشاء جمود (deadlocks). تحدث هذه عندما تحتاج عملية ما إلى قفل موردين (resources) ويكون كل من threadين قد حصل على أحد الـ locks، مما يجعلهما ينتظران بعضهما البعض إلى الأبد. إذا كنت مهتمًا بالـ deadlocks، فحاول إنشاء برنامج Rust يحتوي على deadlock؛ ثم ابحث عن استراتيجيات التخفيف من الـ deadlock لـ mutexes في أي لغة وحاول تطبيقها في Rust. توفر وثائق API لـ `Mutex<T>` و `MutexGuard` في الـ standard library معلومات مفيدة.

سنختتم هذا الفصل بالحديث عن سمات الإرسال والمزامنة (Send and Sync traits) وكيف يمكننا استخدامها مع الـ custom types.

[atomic]: ../std/sync/atomic/index.html
