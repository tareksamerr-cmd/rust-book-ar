
<!-- Old headings. Do not remove or links may break. -->

<a id="yielding"></a>

### التنازل عن التحكم لوقت التشغيل (Yielding Control to the Runtime)

تذكر من قسم ["أول برنامج غير متزامن لنا"][async-program] أنه عند كل نقطة انتظار (await point)، تمنح Rust وقت التشغيل (runtime) فرصة لإيقاف المهمة (task) مؤقتاً والتبديل إلى مهمة أخرى إذا لم يكن الـ future الذي يتم انتظاره جاهزاً. والعكس صحيح أيضاً: Rust تقوم _فقط_ بإيقاف الكتل غير المتزامنة (async blocks) مؤقتاً وتسليم التحكم مرة أخرى إلى الـ runtime عند await point. كل شيء بين await points يكون متزامناً (synchronous).

هذا يعني أنه إذا قمت بالكثير من العمل في async block بدون await point، فإن ذلك الـ future سيمنع أي futures أخرى من إحراز تقدم. قد تسمع أحياناً إشارة إلى هذا على أنه future واحد يقوم بـ _تجويع_ (starving) الـ futures الأخرى. في بعض الحالات، قد لا يكون ذلك أمراً كبيراً. ومع ذلك، إذا كنت تقوم بنوع من الإعداد المكلف أو عمل يستغرق وقتاً طويلاً، أو إذا كان لديك future سيستمر في القيام بمهمة معينة إلى أجل غير مسمى، فستحتاج إلى التفكير في متى وأين تسلم التحكم مرة أخرى إلى الـ runtime.

دعونا نحاكي عملية تستغرق وقتاً طويلاً لتوضيح مشكلة الـ starvation، ثم نستكشف كيفية حلها. تقدم القائمة 17-14 دالة `slow`.

<Listing number="17-14" caption="Using `thread::sleep` to simulate slow operations" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-14/src/main.rs:slow}}
```

</Listing>

يستخدم هذا الكود `std::thread::sleep` بدلاً من `trpl::sleep` بحيث يؤدي استدعاء `slow` إلى حظر (block) الخيط (thread) الحالي لعدد من المللي ثانية. يمكننا استخدام `slow` لتمثيل العمليات الواقعية التي تستغرق وقتاً طويلاً وتكون حظراً (blocking).

في القائمة 17-15، نستخدم `slow` لمحاكاة القيام بهذا النوع من العمل المرتبط بالمعالج (CPU-bound work) في زوج من الـ futures.

<Listing number="17-15" caption="Calling the `slow` function to simulate slow operations" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-15/src/main.rs:slow-futures}}
```

</Listing>

يسلم كل future التحكم مرة أخرى إلى الـ runtime فقط _بعد_ تنفيذ مجموعة من العمليات البطيئة. إذا قمت بتشغيل هذا الكود، فسترى هذا المخرج:

```text
'a' started.
'a' ran for 30ms
'a' ran for 10ms
'a' ran for 20ms
'b' started.
'b' ran for 75ms
'b' ran for 10ms
'b' ran for 15ms
'b' ran for 350ms
'a' finished.
```

كما هو الحال مع القائمة 17-5 حيث استخدمنا `trpl::select` للتسابق بين futures تجلب عنواني URL، لا يزال `select` ينتهي بمجرد انتهاء `a`. ومع ذلك، لا يوجد تداخل (interleaving) بين استدعاءات `slow` في الـ futures الاثنين. يقوم الـ future `a` بكل عمله حتى يتم انتظار استدعاء `trpl::sleep` ، ثم يقوم الـ future `b` بكل عمله حتى يتم انتظار استدعاء `trpl::sleep` الخاص به، وأخيراً يكتمل الـ future `a`. للسماح لكلا الـ futures بإحراز تقدم بين مهامهما البطيئة، نحتاج إلى await points حتى نتمكن من تسليم التحكم مرة أخرى إلى الـ runtime. وهذا يعني أننا بحاجة إلى شيء يمكننا انتظاره!

يمكننا بالفعل رؤية هذا النوع من التسليم يحدث في القائمة 17-15: إذا قمنا بإزالة `trpl::sleep` في نهاية الـ future `a` ، فإنه سيكتمل دون تشغيل الـ future `b` _على الإطلاق_. دعونا نحاول استخدام دالة `trpl::sleep` كنقطة انطلاق للسماح للعمليات بتبادل إحراز التقدم، كما هو موضح في القائمة 17-16.

<Listing number="17-16" caption="Using `trpl::sleep` to let operations switch off making progress" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-16/src/main.rs:here}}
```

</Listing>

لقد أضفنا استدعاءات `trpl::sleep` مع await points بين كل استدعاء لـ `slow`. الآن أصبح عمل الـ futures الاثنين متداخلاً:

```text
'a' started.
'a' ran for 30ms
'b' started.
'b' ran for 75ms
'a' ran for 10ms
'b' ran for 10ms
'a' ran for 20ms
'b' ran for 15ms
'a' finished.
```

لا يزال الـ future `a` يعمل لفترة قبل تسليم التحكم إلى `b` ، لأنه يستدعي `slow` قبل استدعاء `trpl::sleep` على الإطلاق، ولكن بعد ذلك يتبادل الـ futures الأدوار في كل مرة يصل فيها أحدهما إلى await point. في هذه الحالة، قمنا بذلك بعد كل استدعاء لـ `slow` ، ولكن يمكننا تقسيم العمل بأي طريقة نراها منطقية بالنسبة لنا.

نحن لا نريد حقاً أن _ننام_ (sleep) هنا: نريد إحراز تقدم بأسرع ما يمكن. نحتاج فقط إلى إعادة التحكم إلى الـ runtime. يمكننا القيام بذلك مباشرة، باستخدام دالة `trpl::yield_now`. في القائمة 17-17، نستبدل كل استدعاءات `trpl::sleep` تلك بـ `trpl::yield_now`.

<Listing number="17-17" caption="Using `yield_now` to let operations switch off making progress" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-17/src/main.rs:yields}}
```

</Listing>

هذا الكود أكثر وضوحاً بشأن النية الفعلية ويمكن أن يكون أسرع بكثير من استخدام `sleep` ، لأن المؤقتات (timers) مثل تلك المستخدمة بواسطة `sleep` غالباً ما يكون لها حدود لمدى دقتها. نسخة `sleep` التي نستخدمها، على سبيل المثال، ستنام دائماً لمدة مللي ثانية واحدة على الأقل، حتى لو مررنا لها مدة (Duration) تبلغ نانو ثانية واحدة. مرة أخرى، أجهزة الكمبيوتر الحديثة _سريعة_: يمكنها القيام بالكثير في مللي ثانية واحدة!

هذا يعني أن async يمكن أن يكون مفيداً حتى للمهام المرتبطة بالحساب (compute-bound tasks)، اعتماداً على ما يفعله برنامجك أيضاً، لأنه يوفر أداة مفيدة لهيكلة العلاقات بين أجزاء مختلفة من البرنامج (ولكن بتكلفة العبء الإضافي لآلة الحالة غير المتزامنة (async state machine)). هذا شكل من أشكال _تعدد المهام التعاوني_ (cooperative multitasking)، حيث يمتلك كل future القدرة على تحديد متى يسلم التحكم عبر await points. لذلك يتحمل كل future أيضاً مسؤولية تجنب الحظر لفترة طويلة جداً. في بعض أنظمة التشغيل المدمجة (embedded operating systems) القائمة على Rust، هذا هو النوع _الوحيد_ من تعدد المهام!

في الكود الواقعي، لن تقوم عادةً بتبديل استدعاءات الدوال مع await points في كل سطر بالطبع. في حين أن التنازل عن التحكم بهذه الطريقة غير مكلف نسبياً، إلا أنه ليس مجانياً. في كثير من الحالات، قد يؤدي محاولة تقسيم مهمة مرتبطة بالحساب إلى جعلها أبطأ بشكل ملحوظ، لذا أحياناً يكون من الأفضل للأداء _العام_ السماح لعملية ما بالحظر لفترة وجيزة. قم دائماً بالقياس لمعرفة ماهية اختناقات الأداء الفعلية في كودك. ومع ذلك، من المهم إبقاء الديناميكية الأساسية في الاعتبار إذا كنت _ترى_ الكثير من العمل يحدث بشكل تسلسلي وكنت تتوقع حدوثه بشكل متزامن (concurrently)!

### بناء تجريداتنا غير المتزامنة الخاصة (Building Our Own Async Abstractions)

يمكننا أيضاً تركيب الـ futures معاً لإنشاء أنماط جديدة. على سبيل المثال، يمكننا بناء دالة `timeout` باستخدام لبنات بناء غير متزامنة لدينا بالفعل. عندما ننتهي، ستكون النتيجة لبنة بناء أخرى يمكننا استخدامها لإنشاء المزيد من التجريدات غير المتزامنة (async abstractions).

توضح القائمة 17-18 كيف نتوقع أن يعمل `timeout` هذا مع future بطيء.

<Listing number="17-18" caption="Using our imagined `timeout` to run a slow operation with a time limit" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-18/src/main.rs:here}}
```

</Listing>

دعونا ننفذ هذا! للبدء، دعونا نفكر في واجهة برمجة التطبيقات (API) لـ `timeout`:

- يجب أن تكون دالة غير متزامنة (async function) نفسها حتى نتمكن من انتظارها.
- يجب أن يكون معاملها (parameter) الأول هو future للتشغيل. يمكننا جعله عاماً (generic) للسماح له بالعمل مع أي future.
- سيكون parameter الثاني هو أقصى وقت للانتظار. إذا استخدمنا `Duration` ، فسيجعل ذلك من السهل تمريره إلى `trpl::sleep`.
- يجب أن تعيد `Result`. إذا اكتمل الـ future بنجاح، فسيكون الـ `Result` هو `Ok` مع القيمة التي أنتجها الـ future. إذا انقضت مهلة الانتظار أولاً، فسيكون الـ `Result` هو `Err` مع المدة التي انتظرها الـ timeout.

توضح القائمة 17-19 هذا التصريح.

<Listing number="17-19" caption="Defining the signature of `timeout`" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-19/src/main.rs:declaration}}
```

</Listing>

هذا يلبي أهدافنا بالنسبة للأنواع. الآن دعونا نفكر في _السلوك_ الذي نحتاجه: نريد تسابق الـ future الممرر مقابل المدة. يمكننا استخدام `trpl::sleep` لإنشاء future مؤقت (timer future) من المدة، واستخدام `trpl::select` لتشغيل ذلك الـ timer مع الـ future الذي يمرره المستدعي.

في القائمة 17-20، ننفذ `timeout` عن طريق المطابقة (matching) على نتيجة انتظار `trpl::select`.

<Listing number="17-20" caption="Defining `timeout` with `select` and `sleep`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-20/src/main.rs:implementation}}
```

</Listing>

تنفيذ `trpl::select` ليس عادلاً: فهو يقوم دائماً بـ polling للمعاملات بالترتيب الذي تم تمريرها به (تنفيذات `select` الأخرى ستختار عشوائياً أي معامل ستقوم بـ polling له أولاً). وبالتالي، نمرر `future_to_try` إلى `select` أولاً حتى يحصل على فرصة للاكتمال حتى لو كان `max_time` مدة قصيرة جداً. إذا انتهى `future_to_try` أولاً، فسيعيد `select` القيمة `Left` مع مخرجات `future_to_try`. إذا انتهى الـ `timer` أولاً، فسيعيد `select` القيمة `Right` مع مخرجات الـ timer وهي `()`.

إذا نجح `future_to_try` وحصلنا على `Left(output)` ، فإننا نعيد `Ok(output)`. إذا انقضى مؤقت النوم بدلاً من ذلك وحصلنا على `Right(())` ، فإننا نتجاهل الـ `()` باستخدام `_` ونعيد `Err(max_time)` بدلاً من ذلك.

بذلك، لدينا `timeout` يعمل مبني من مساعدين غير متزامنين آخرين. إذا قمنا بتشغيل الكود الخاص بنا، فسيطبع وضع الفشل بعد المهلة:

```text
Failed after 2 seconds
```

لأن الـ futures تتركب مع futures أخرى، يمكنك بناء أدوات قوية حقاً باستخدام لبنات بناء غير متزامنة أصغر. على سبيل المثال، يمكنك استخدام نفس هذا النهج لدمج المهلات (timeouts) مع عمليات إعادة المحاولة (retries)، واستخدامها بدورها مع عمليات مثل استدعاءات الشبكة (مثل تلك الموجودة في القائمة 17-5).

في الممارسة العملية، ستعمل عادةً مباشرة مع `async` و `await` ، وبشكل ثانوي مع دوال مثل `select` وماكروهات مثل ماكرو `join!` للتحكم في كيفية تنفيذ الـ futures الخارجية.

لقد رأينا الآن عدداً من الطرق للعمل مع عدة futures في نفس الوقت. بعد ذلك، سنلقي نظرة على كيفية العمل مع عدة futures في تسلسل عبر الزمن باستخدام _التدفقات_ (streams).

[async-program]: ch17-01-futures-and-syntax.html#our-first-async-program
```
