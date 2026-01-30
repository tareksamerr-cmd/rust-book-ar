<!-- Old headings. Do not remove or links may break. -->

<a id="using-message-passing-to-transfer-data-between-threads"></a>

## نقل البيانات بين الخيوط (Threads) باستخدام تمرير الرسائل (Message Passing)

أحد الأساليب الشائعة بشكل متزايد لضمان التزامن (concurrency) الآمن هو *تمرير الرسائل* (message passing)، حيث تتواصل الـ threads أو الـ actors عن طريق إرسال رسائل تحتوي على بيانات لبعضها البعض. إليك الفكرة في شعار من [وثائق لغة Go](https://golang.org/doc/effective_go.html#concurrency):
> "لا تتواصل عن طريق مشاركة الذاكرة (sharing memory)؛ بدلاً من ذلك، شارك الذاكرة عن طريق التواصل."

لتحقيق تزامن إرسال الرسائل، توفر الـ standard library في Rust تطبيقًا لـ *القنوات* (channels). الـ channel هو مفهوم برمجي عام يتم من خلاله إرسال البيانات من thread إلى آخر.

يمكنك تخيل الـ channel في البرمجة على أنه يشبه قناة مائية اتجاهية، مثل جدول أو نهر. إذا وضعت شيئًا مثل بطة مطاطية في نهر، فسوف تنتقل إلى أسفل النهر حتى نهاية المجرى المائي.

يحتوي الـ channel على نصفين: *مرسل* (transmitter) و *مستقبل* (receiver). نصف الـ transmitter هو الموقع العلوي حيث تضع البطة المطاطية في النهر، ونصف الـ receiver هو المكان الذي تنتهي فيه البطة المطاطية في المصب. يستدعي جزء واحد من الكود الخاص بك methods على الـ transmitter بالبيانات التي تريد إرسالها، ويتحقق جزء آخر من طرف الـ receiving بحثًا عن الرسائل الواردة. يقال إن الـ channel *مغلق* (closed) إذا تم إسقاط (dropped) أي من نصفي الـ transmitter أو الـ receiver.

هنا، سنعمل على برنامج يحتوي على thread واحد لإنشاء قيم وإرسالها عبر channel، و thread آخر سيستقبل القيم ويطبعها. سنرسل قيمًا بسيطة بين الـ threads باستخدام channel لتوضيح الميزة. بمجرد أن تكون على دراية بالتقنية، يمكنك استخدام الـ channels لأي threads تحتاج إلى التواصل مع بعضها البعض، مثل نظام دردشة أو نظام تقوم فيه threads متعددة بأداء أجزاء من عملية حسابية وإرسال الأجزاء إلى thread واحد يجمع النتائج.

أولاً، في القائمة 16-6، سننشئ channel ولكن لن نفعل به أي شيء. لاحظ أن هذا لن يتم تجميعه بعد لأن Rust لا يمكنها معرفة نوع القيم التي نريد إرسالها عبر الـ channel.

<Listing number="16-6" file-name="src/main.rs" caption="إنشاء channel وتعيين النصفين لـ `tx` و `rx`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-06/src/main.rs}}
```

</Listing>

ننشئ channel جديدًا باستخدام دالة `mpsc::channel`؛ يشير `mpsc` إلى *منتج متعدد، مستهلك واحد* (multiple producer, single consumer). باختصار، الطريقة التي تطبق بها الـ standard library في Rust الـ channels تعني أن الـ channel يمكن أن يحتوي على أطراف *إرسال* (sending) متعددة تنتج قيمًا ولكن طرف *استقبال* (receiving) واحد فقط يستهلك تلك القيم. تخيل جداول متعددة تتدفق معًا في نهر واحد كبير: كل ما يتم إرساله عبر أي من الجداول سينتهي به المطاف في نهر واحد في النهاية. سنبدأ بـ single producer في الوقت الحالي، ولكننا سنضيف multiple producers عندما نجعل هذا المثال يعمل.

تُرجع دالة `mpsc::channel` tuple، العنصر الأول منها هو طرف الـ sending - الـ transmitter - والعنصر الثاني هو طرف الـ receiving - الـ receiver. تُستخدم الاختصارات `tx` و `rx` تقليديًا في العديد من المجالات لـ *transmitter* و *receiver*، على التوالي، لذلك نسمي متغيراتنا على هذا النحو للإشارة إلى كل طرف. نحن نستخدم عبارة `let` بنمط (pattern) يفكك الـ tuples؛ سنناقش استخدام الـ patterns في عبارات `let` والـ destructuring في الفصل 19. في الوقت الحالي، اعلم أن استخدام عبارة `let` بهذه الطريقة هو أسلوب مناسب لاستخراج أجزاء الـ tuple التي تُرجعها `mpsc::channel`.

دعنا ننقل طرف الـ transmitting إلى thread تم إنشاؤه حديثًا ونجعله يرسل string واحدًا بحيث يتواصل الـ thread الذي تم إنشاؤه مع الـ main thread، كما هو موضح في القائمة 16-7. هذا يشبه وضع بطة مطاطية في النهر في المنبع أو إرسال رسالة دردشة من thread إلى آخر.

<Listing number="16-7" file-name="src/main.rs" caption='نقل `tx` إلى thread تم إنشاؤه وإرسال `"hi"`'>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-07/src/main.rs}}
```

</Listing>

مرة أخرى، نستخدم `thread::spawn` لإنشاء thread جديد ثم نستخدم `move` لنقل `tx` إلى الـ closure بحيث يمتلك الـ thread الذي تم إنشاؤه الـ `tx`. يحتاج الـ thread الذي تم إنشاؤه إلى امتلاك الـ transmitter ليتمكن من إرسال الرسائل عبر الـ channel.

يحتوي الـ transmitter على method `send` الذي يأخذ القيمة التي نريد إرسالها. يُرجع method `send` نوع `Result<T, E>`، لذلك إذا تم إسقاط الـ receiver بالفعل ولم يكن هناك مكان لإرسال قيمة، فستُرجع عملية الـ send خطأ. في هذا المثال، نستدعي `unwrap` للإصابة بالذعر في حالة حدوث خطأ. ولكن في تطبيق حقيقي، سنتعامل معه بشكل صحيح: ارجع إلى الفصل 9 لمراجعة استراتيجيات معالجة الأخطاء المناسبة.

في القائمة 16-8، سنحصل على القيمة من الـ receiver في الـ main thread. هذا يشبه استرداد البطة المطاطية من الماء في نهاية النهر أو تلقي رسالة دردشة.

<Listing number="16-8" file-name="src/main.rs" caption='استقبال القيمة `"hi"` في الـ main thread وطباعتها'>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-08/src/main.rs}}
```

</Listing>

يحتوي الـ receiver على methodين مفيدين: `recv` و `try_recv`. نحن نستخدم `recv`، وهو اختصار لـ *receive* (استقبال)، والذي سيحظر تنفيذ الـ main thread وينتظر حتى يتم إرسال قيمة عبر الـ channel. بمجرد إرسال قيمة، ستُرجعها `recv` في `Result<T, E>`. عندما يغلق الـ transmitter، ستُرجع `recv` خطأ للإشارة إلى أنه لن تأتي المزيد من القيم.

لا يحظر method `try_recv`، ولكنه بدلاً من ذلك سيُرجع `Result<T, E>` على الفور: قيمة `Ok` تحتوي على رسالة إذا كانت متوفرة وقيمة `Err` إذا لم تكن هناك أي رسائل هذه المرة. يعد استخدام `try_recv` مفيدًا إذا كان هذا الـ thread لديه عمل آخر للقيام به أثناء انتظار الرسائل: يمكننا كتابة حلقة تستدعي `try_recv` كل فترة، وتتعامل مع رسالة إذا كانت متوفرة، وإلا فإنها تقوم بعمل آخر لفترة قصيرة حتى تتحقق مرة أخرى.

لقد استخدمنا `recv` في هذا المثال للتبسيط؛ ليس لدينا أي عمل آخر لـ main thread للقيام به بخلاف انتظار الرسائل، لذا فإن حظر الـ main thread مناسب.

عندما نقوم بتشغيل الكود في القائمة 16-8، سنرى القيمة المطبوعة من الـ main thread:

```text
Got: hi
```

ممتاز!

<!-- Old headings. Do not remove or links may break. -->

<a id="channels-and-ownership-transference"></a>

### نقل الملكية (Ownership) عبر القنوات

تلعب قواعد الـ ownership دورًا حيويًا في message sending لأنها تساعدك على كتابة كود تزامن آمن. يعد منع الأخطاء في البرمجة المتزامنة ميزة التفكير في الـ ownership في جميع برامج Rust الخاصة بك. دعنا نجري تجربة لإظهار كيف يعمل الـ channels والـ ownership معًا لمنع المشاكل: سنحاول استخدام قيمة `val` في الـ thread الذي تم إنشاؤه *بعد* أن أرسلناها عبر الـ channel. حاول تجميع الكود في القائمة 16-9 لترى لماذا لا يُسمح بهذا الكود.

<Listing number="16-9" file-name="src/main.rs" caption="محاولة استخدام `val` بعد أن أرسلناها عبر الـ channel">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-09/src/main.rs}}
```

</Listing>

هنا، نحاول طباعة `val` بعد أن أرسلناها عبر الـ channel عبر `tx.send`. السماح بذلك سيكون فكرة سيئة: بمجرد إرسال القيمة إلى thread آخر، يمكن لهذا الـ thread تعديلها أو إسقاطها قبل أن نحاول استخدام القيمة مرة أخرى. من المحتمل أن تتسبب تعديلات الـ thread الآخر في حدوث أخطاء أو نتائج غير متوقعة بسبب بيانات غير متسقة أو غير موجودة. ومع ذلك، تعطينا Rust خطأ إذا حاولنا تجميع الكود في القائمة 16-9:

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-09/output.txt}}
```

لقد تسبب خطأ التزامن الخاص بنا في حدوث خطأ في وقت التجميع (compile-time error). تأخذ دالة `send` الـ ownership لـ parameter الخاص بها، وعندما يتم نقل القيمة، يأخذ الـ receiver الـ ownership لها. هذا يمنعنا من استخدام القيمة مرة أخرى عن طريق الخطأ بعد إرسالها؛ يتحقق نظام الـ ownership من أن كل شيء على ما يرام.

<!-- Old headings. Do not remove or links may break. -->

<a id="sending-multiple-values-and-seeing-the-receiver-waiting"></a>

### إرسال قيم متعددة

تم تجميع وتشغيل الكود في القائمة 16-8، ولكنه لم يوضح لنا بوضوح أن threadين منفصلين كانا يتحدثان مع بعضهما البعض عبر الـ channel.

في القائمة 16-10، أجرينا بعض التعديلات التي ستثبت أن الكود في القائمة 16-8 يعمل بشكل متزامن (concurrently): سيرسل الـ thread الذي تم إنشاؤه الآن رسائل متعددة ويتوقف مؤقتًا لمدة ثانية بين كل رسالة.

<Listing number="16-10" file-name="src/main.rs" caption="إرسال رسائل متعددة والتوقف مؤقتًا بين كل واحدة">

```rust,noplayground
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-10/src/main.rs}}
```

</Listing>

هذه المرة، يحتوي الـ thread الذي تم إنشاؤه على vector من الـ strings التي نريد إرسالها إلى الـ main thread. نكرر عليها، ونرسل كل واحدة على حدة، ونتوقف مؤقتًا بين كل واحدة عن طريق استدعاء دالة `thread::sleep` بقيمة `Duration` تبلغ ثانية واحدة.

في الـ main thread، لم نعد نستدعي دالة `recv` بشكل صريح: بدلاً من ذلك، نتعامل مع `rx` كـ *مكرر* (iterator). لكل قيمة يتم استقبالها، نقوم بطباعتها. عندما يتم إغلاق الـ channel، سينتهي التكرار.

عند تشغيل الكود في القائمة 16-10، يجب أن ترى الإخراج التالي مع توقف لمدة ثانية واحدة بين كل سطر:

```text
Got: hi
Got: from
Got: the
Got: thread
```

نظرًا لعدم وجود أي كود يتوقف مؤقتًا أو يتأخر في حلقة `for` في الـ main thread، يمكننا أن نقول إن الـ main thread ينتظر استقبال القيم من الـ thread الذي تم إنشاؤه.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-multiple-producers-by-cloning-the-transmitter"></a>

### إنشاء منتجين متعددين (Multiple Producers)

في وقت سابق ذكرنا أن `mpsc` هو اختصار لـ *multiple producer, single consumer*. دعنا نستخدم `mpsc` ونوسع الكود في القائمة 16-10 لإنشاء threads متعددة ترسل جميعها قيمًا إلى نفس الـ receiver. يمكننا القيام بذلك عن طريق استنساخ (cloning) الـ transmitter، كما هو موضح في القائمة 16-11.

<Listing number="16-11" file-name="src/main.rs" caption="إرسال رسائل متعددة من multiple producers">

```rust,noplayground
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-11/src/main.rs:here}}
```

</Listing>

هذه المرة، قبل أن ننشئ الـ thread الأول، نستدعي `clone` على الـ transmitter. سيعطينا هذا transmitter جديدًا يمكننا تمريره إلى الـ thread الأول الذي تم إنشاؤه. نمرر الـ transmitter الأصلي إلى thread ثانٍ تم إنشاؤه. هذا يعطينا threadين، يرسل كل منهما رسائل مختلفة إلى الـ receiver الواحد.

عند تشغيل الكود، يجب أن يبدو الإخراج الخاص بك شيئًا كهذا:

```text
Got: hi
Got: more
Got: from
Got: messages
Got: for
Got: the
Got: thread
Got: you
```

قد ترى القيم بترتيب آخر، اعتمادًا على نظامك. هذا ما يجعل الـ concurrency مثيرًا للاهتمام وصعبًا أيضًا. إذا قمت بالتجربة باستخدام `thread::sleep`، وإعطائه قيمًا مختلفة في الـ threads المختلفة، فسيكون كل تشغيل غير حتمي (nondeterministic) بشكل أكبر وينتج إخراجًا مختلفًا في كل مرة.

الآن بعد أن نظرنا في كيفية عمل الـ channels، دعنا نلقي نظرة على طريقة مختلفة لـ concurrency.
