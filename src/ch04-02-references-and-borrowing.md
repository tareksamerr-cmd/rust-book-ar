## المراجع والاستعارة (References and Borrowing)

المشكلة في كود الصفوف (tuple) في القائمة 4-5 هي أننا نضطر إلى إعادة `String` إلى الدالة المستدعية حتى نتمكن من الاستمرار في استخدام `String` بعد استدعاء `calculate_length` ، لأن `String` تم نقله (moved) إلى `calculate_length`. بدلاً من ذلك، يمكننا تقديم مرجع (reference) لقيمة `String`. المرجع يشبه المؤشر (pointer) في أنه عنوان يمكننا اتباعه للوصول إلى البيانات المخزنة في ذلك العنوان؛ تلك البيانات مملوكة لمتغير آخر. على عكس pointer، يضمن المرجع أن يشير إلى قيمة صالحة من نوع معين طوال فترة حياة ذلك المرجع.

إليك كيف يمكنك تعريف واستخدام دالة `calculate_length` التي تحتوي على مرجع لكائن كمعلمة (parameter) بدلاً من أخذ ملكية القيمة:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-07-reference/src/main.rs:all}}
```

</Listing>

أولاً، لاحظ أن كل كود tuple في تصريح المتغير وقيمة إرجاع الدالة قد اختفى. ثانياً، لاحظ أننا نمرر `&s1` إلى `calculate_length` وفي تعريفها، نأخذ `&String` بدلاً من `String`. تمثل علامات الاند (ampersands) هذه المراجع، وهي تسمح لك بالإشارة إلى قيمة ما دون أخذ ملكيتها. يوضح الشكل 4-6 هذا المفهوم.

<img alt="Three tables: the table for s contains only a pointer to the table
for s1. The table for s1 contains the stack data for s1 and points to the
string data on the heap." src="img/trpl04-06.svg" class="center" />

<span class="caption">الشكل 4-6: مخطط لـ `&String` `s` يشير إلى `String` `s1`</span>

> ملاحظة: عكس عملية الإسناد المرجعي (referencing) باستخدام `&` هو _إلغاء المرجعية_ (dereferencing)، والذي يتم تحقيقه باستخدام عامل إلغاء المرجعية (dereference operator) `*`. سنرى بعض استخدامات dereference operator في الفصل 8 ونناقش تفاصيل dereferencing في الفصل 15.

دعونا نلقي نظرة فاحصة على استدعاء الدالة هنا:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-07-reference/src/main.rs:here}}
```

يسمح لنا بناء جملة `&s1` بإنشاء مرجع _يشير_ إلى قيمة `s1` ولكنه لا يملكها. ولأن المرجع لا يملكها، فإن القيمة التي يشير إليها لن يتم حذفها (dropped) عندما يتوقف استخدام المرجع.

وبالمثل، يستخدم توقيع الدالة `&` للإشارة إلى أن نوع parameter `s` هو مرجع. دعونا نضيف بعض التوضيحات الشارحة:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-08-reference-with-annotations/src/main.rs:here}}
```

النطاق (scope) الذي يكون فيه المتغير `s` صالحاً هو نفس نطاق أي parameter للدالة، ولكن القيمة التي يشير إليها المرجع لا يتم حذفها عندما يتوقف استخدام `s` ، لأن `s` لا يملك الملكية. عندما تحتوي الدوال على مراجع كمعلمات بدلاً من القيم الفعلية، فلن نحتاج إلى إعادة القيم من أجل إعادة الملكية، لأننا لم نمتلك الملكية أبداً.

نسمي عملية إنشاء مرجع _استعارة_ (borrowing). كما هو الحال في الحياة الواقعية، إذا كان شخص ما يملك شيئاً ما، يمكنك استعارته منه. عندما تنتهي، عليك إعادته. أنت لا تملكه.

لذا، ماذا يحدث إذا حاولنا تعديل شيء نستعيره؟ جرب الكود في القائمة 4-6. تنبيه: إنه لا يعمل!

<Listing number="4-6" file-name="src/main.rs" caption="محاولة تعديل قيمة مستعارة">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-06/src/main.rs}}
```

</Listing>

إليك الخطأ:

```console
{{#include ../listings/ch04-understanding-ownership/listing-04-06/output.txt}}
```

تماماً كما أن المتغيرات غير قابلة للتغيير (immutable) بشكل افتراضي، كذلك المراجع. لا يُسمح لنا بتعديل شيء لدينا مرجع له.

### المراجع القابلة للتغيير (Mutable References)

يمكننا إصلاح الكود من القائمة 4-6 للسماح لنا بتعديل قيمة مستعارة ببعض التعديلات الصغيرة التي تستخدم، بدلاً من ذلك، _مرجعاً قابلاً للتغيير_ (mutable reference):

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-09-fixes-listing-04-06/src/main.rs}}
```

</Listing>

أولاً، نقوم بتغيير `s` ليكون `mut`. بعد ذلك، ننشئ mutable reference باستخدام `&mut s` حيث نستدعي دالة `change` ونقوم بتحديث توقيع الدالة لقبول mutable reference باستخدام `some_string: &mut String`. هذا يجعل من الواضح جداً أن دالة `change` ستقوم بتغيير (mutate) القيمة التي تستعيرها.

المراجع القابلة للتغيير لها قيد واحد كبير: إذا كان لديك mutable reference لقيمة ما، فلا يمكن أن يكون لديك مراجع أخرى لتلك القيمة. هذا الكود الذي يحاول إنشاء اثنين من mutable references لـ `s` سيفشل:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-10-multiple-mut-not-allowed/src/main.rs:here}}
```

</Listing>

إليك الخطأ:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-10-multiple-mut-not-allowed/output.txt}}
```

يقول هذا الخطأ أن هذا الكود غير صالح لأننا لا نستطيع استعارة `s` كـ mutable أكثر من مرة في كل مرة. الاستعارة القابلة للتغيير الأولى موجودة في `r1` ويجب أن تستمر حتى يتم استخدامها في `println!` ، ولكن بين إنشاء ذلك المرجع القابل للتغيير واستخدامه، حاولنا إنشاء mutable reference آخر في `r2` يستعير نفس البيانات مثل `r1`.

القيد الذي يمنع وجود مراجع متعددة قابلة للتغيير لنفس البيانات في نفس الوقت يسمح بالتغيير ولكن بطريقة محكومة للغاية. إنه شيء يعاني منه الـ Rustaceans الجدد لأن معظم اللغات تسمح لك بالتغيير وقتما تشاء. الفائدة من وجود هذا القيد هي أن Rust يمكنها منع سباقات البيانات (data races) في وقت التصريف (compile time). _سباق البيانات_ يشبه حالة السباق (race condition) ويحدث عند وقوع هذه السلوكيات الثلاثة:

- وصول اثنين أو أكثر من pointers إلى نفس البيانات في نفس الوقت.
- استخدام واحد على الأقل من pointers للكتابة في البيانات.
- عدم وجود آلية مستخدمة لمزامنة الوصول إلى البيانات.

تسبب سباقات البيانات سلوكاً غير محدد (undefined behavior) ويمكن أن يكون من الصعب تشخيصها وإصلاحها عندما تحاول تعقبها في وقت التشغيل (runtime)؛ تمنع Rust هذه المشكلة برفض تصريف الكود الذي يحتوي على data races!

كما هو الحال دائماً، يمكننا استخدام الأقواس المتعرجة لإنشاء scope جديد، مما يسمح بوجود مراجع متعددة قابلة للتغيير، ولكن ليس مراجع _متزامنة_ (simultaneous):

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-11-muts-in-separate-scopes/src/main.rs:here}}
```

تفرض Rust قاعدة مماثلة للجمع بين المراجع القابلة للتغيير وغير القابلة للتغيير. يؤدي هذا الكود إلى حدوث خطأ:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-12-immutable-and-mutable-not-allowed/src/main.rs:here}}
```

إليك الخطأ:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-12-immutable-and-mutable-not-allowed/output.txt}}
```

يا للهول! لا يمكننا _أيضاً_ الحصول على mutable reference بينما لدينا مرجع غير قابل للتغيير لنفس القيمة.

لا يتوقع مستخدمو المرجع غير القابل للتغيير أن تتغير القيمة فجأة من تحتهم! ومع ذلك، يُسمح بوجود مراجع متعددة غير قابلة للتغيير لأن لا أحد يقرأ البيانات فقط لديه القدرة على التأثير على قراءة أي شخص آخر للبيانات.

لاحظ أن scope المرجع يبدأ من حيث يتم تقديمه ويستمر حتى آخر مرة يتم فيها استخدام ذلك المرجع. على سبيل المثال، سيتم تصريف هذا الكود لأن آخر استخدام للمراجع غير القابلة للتغيير هو في `println!` ، قبل تقديم mutable reference:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-13-reference-scope-ends/src/main.rs:here}}
```

تنتهي نطاقات المراجع غير القابلة للتغيير `r1` و `r2` بعد `println!` حيث تم استخدامهما لآخر مرة، وهو ما يحدث قبل إنشاء mutable reference `r3`. هذه النطاقات لا تتداخل، لذا فإن هذا الكود مسموح به: يمكن لـ compiler معرفة أن المرجع لم يعد مستخدماً عند نقطة قبل نهاية scope.

على الرغم من أن أخطاء borrowing قد تكون محبطة في بعض الأحيان، تذكر أن compiler في Rust هو من يشير إلى خطأ محتمل في وقت مبكر (في compile time بدلاً من runtime) ويوضح لك بالضبط مكان المشكلة. عندها، لن تضطر إلى تعقب سبب عدم كون بياناتك كما كنت تعتقد.

### المراجع المعلقة (Dangling References)

في اللغات التي تحتوي على pointers، من السهل إنشاء _مؤشر معلق_ (dangling pointer) عن طريق الخطأ — وهو مؤشر يشير إلى موقع في الذاكرة ربما تم إعطاؤه لشخص آخر — عن طريق تحرير بعض الذاكرة مع الاحتفاظ بمؤشر لتلك الذاكرة. في Rust، على النقيض من ذلك، يضمن compiler أن المراجع لن تكون أبداً مراجع معلقة (dangling references): إذا كان لديك مرجع لبعض البيانات، فسيضمن compiler أن البيانات لن تخرج عن النطاق قبل أن يخرج المرجع للبيانات عن النطاق.

دعونا نحاول إنشاء dangling reference لنرى كيف تمنعها Rust بخطأ في وقت التصريف:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-14-dangling-reference/src/main.rs}}
```

</Listing>

إليك الخطأ:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-14-dangling-reference/output.txt}}
```

تشير رسالة الخطأ هذه إلى ميزة لم نغطها بعد: فترات الحياة (lifetimes). سنناقش lifetimes بالتفصيل في الفصل 10. ولكن، إذا تجاهلت الأجزاء المتعلقة بـ lifetimes، فإن الرسالة تحتوي بالفعل على المفتاح لسبب كون هذا الكود مشكلة:

```text
this function's return type contains a borrowed value, but there is no value
for it to be borrowed from
```

دعونا نلقي نظرة فاحصة على ما يحدث بالضبط في كل مرحلة من مراحل كود `dangle` الخاص بنا:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-15-dangling-reference-annotated/src/main.rs:here}}
```

</Listing>

لأن `s` تم إنشاؤه داخل `dangle` ، فعند انتهاء كود `dangle` ، سيتم إلغاء تخصيص (deallocated) `s`. لكننا حاولنا إعادة مرجع له. هذا يعني أن هذا المرجع سيشير إلى `String` غير صالح. هذا ليس جيداً! لن تسمح لنا Rust بالقيام بذلك.

الحل هنا هو إعادة `String` مباشرة:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-16-no-dangle/src/main.rs:here}}
```

هذا يعمل دون أي مشاكل. يتم نقل الملكية للخارج، ولا يتم إلغاء تخصيص أي شيء.

### قواعد المراجع (The Rules of References)

دعونا نلخص ما ناقشناه حول المراجع:

- في أي وقت معين، يمكنك الحصول على _إما_ مرجع واحد قابل للتغيير _أو_ أي عدد من المراجع غير القابلة للتغيير.
- يجب أن تكون المراجع صالحة دائماً.

بعد ذلك، سننظر في نوع مختلف من المراجع: الشرائح (slices).
