## جميع الأماكن التي يمكن استخدام الأنماط (Patterns) فيها

تظهر الأنماط (Patterns) في عدد من الأماكن في Rust، وكنت تستخدمها كثيرًا دون أن تدرك ذلك! يناقش هذا القسم جميع الأماكن التي تكون فيها Patterns صالحة.

### أذرع `match` (match arms)

كما نوقش في الفصل 6، نستخدم Patterns في أذرع `match` (match arms) لتعبيرات `match` (match expressions). رسميًا، يتم تعريف تعبيرات `match` على أنها الكلمة المفتاحية `match`، وقيمة للمطابقة عليها، وواحد أو أكثر من match arms التي تتكون من نمط (pattern) وتعبير (expression) يتم تشغيله إذا كانت القيمة تطابق pattern لهذا الذراع، مثل هذا:

<!--
  Manually formatted rather than using Markdown intentionally: Markdown does not
  support italicizing code in the body of a block like this!
-->

<pre><code>match <em>VALUE</em> {
    <em>PATTERN</em> => <em>EXPRESSION</em>,
    <em>PATTERN</em> => <em>EXPRESSION</em>,
    <em>PATTERN</em> => <em>EXPRESSION</em>,
}</code></pre>

على سبيل المثال، إليك تعبير `match` من القائمة 6-5 الذي يطابق قيمة `Option<i32>` في المتغير (variable) `x`:

```rust,ignore
match x {
    None => None,
    Some(i) => Some(i + 1),
}
```

الـ Patterns في تعبير `match` هذا هي `None` و `Some(i)` على يسار كل سهم.

أحد متطلبات تعبيرات `match` هو أنها يجب أن تكون شاملة (exhaustive) بمعنى أنه يجب أخذ جميع الاحتمالات للقيمة في تعبير `match` في الاعتبار. إحدى الطرق لضمان تغطية كل الاحتمالات هي أن يكون لديك نمط شامل (catch-all pattern) للذراع الأخير: على سبيل المثال، اسم variable يطابق أي قيمة لا يمكن أن يفشل أبدًا وبالتالي يغطي كل حالة متبقية.

الـ pattern المحدد `_` سيطابق أي شيء، ولكنه لا يرتبط أبدًا بـ variable، لذلك غالبًا ما يستخدم في ذراع match الأخير. يمكن أن يكون pattern `_` مفيدًا عندما تريد تجاهل أي قيمة غير محددة، على سبيل المثال. سنغطي pattern `_` بمزيد من التفصيل في [“Ignoring Values in a Pattern”][ignoring-values-in-a-pattern]<!-- ignore --> لاحقًا في هذا الفصل.

### عبارات `let` (let statements)

قبل هذا الفصل، لم نناقش صراحةً استخدام Patterns إلا مع `match` و `if let`، ولكن في الواقع، استخدمنا Patterns في أماكن أخرى أيضًا، بما في ذلك في let statements. على سبيل المثال، ضع في اعتبارك تعيين متغير (variable assignment) المباشر هذا باستخدام `let`:

```rust
let x = 5;
```

في كل مرة استخدمت فيها let statement كهذا، كنت تستخدم Patterns، على الرغم من أنك ربما لم تدرك ذلك! بشكل أكثر رسمية، تبدو let statement كما يلي:

<!--
  Manually formatted rather than using Markdown intentionally: Markdown does not
  support italicizing code in the body of a block like this!
-->

<pre>
<code>let <em>PATTERN</em> = <em>EXPRESSION</em>;</code>
</pre>

في عبارات مثل `let x = 5;` مع اسم variable في خانة PATTERN، فإن اسم variable هو مجرد شكل بسيط بشكل خاص من pattern. يقارن Rust الـ expression بـ pattern ويعين أي أسماء يجدها. لذلك، في مثال `let x = 5;`، فإن `x` هو pattern يعني "اربط ما يطابق هنا بـ variable `x`." نظرًا لأن اسم `x` هو pattern بأكمله، فإن هذا pattern يعني فعليًا "اربط كل شيء بـ variable `x`، مهما كانت القيمة."

لرؤية جانب مطابقة الـ pattern لـ `let` بشكل أكثر وضوحًا، ضع في اعتبارك القائمة 19-1، التي تستخدم pattern مع `let` لـ تفكيك (destructure) صف (tuple).

<Listing number="19-1" caption="Using a pattern to destructure a tuple and create three variables at once">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-01/src/main.rs:here}}
```

</Listing>

هنا، نطابق tuple بـ pattern. يقارن Rust القيمة `(1, 2, 3)` بـ pattern `(x, y, z)` ويرى أن القيمة تطابق pattern - أي أنه يرى أن عدد العناصر هو نفسه في كليهما - لذلك يربط Rust `1` بـ `x`، و `2` بـ `y`، و `3` بـ `z`. يمكنك التفكير في pattern tuple هذا على أنه يتداخل فيه ثلاثة Patterns variable فردية.

إذا كان عدد العناصر في pattern لا يطابق عدد العناصر في tuple، فلن يتطابق النوع الكلي وسنحصل على خطأ المترجم (compiler error). على سبيل المثال، توضح القائمة 19-2 محاولة لـ destructure tuple بثلاثة عناصر إلى اثنين variables، وهو ما لن ينجح.

<Listing number="19-2" caption="Incorrectly constructing a pattern whose variables don’t match the number of elements in the tuple">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-02/src/main.rs:here}}
```

</Listing>

تؤدي محاولة تجميع هذا الكود إلى خطأ النوع هذا:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-02/output.txt}}
```

لإصلاح الـ error، يمكننا تجاهل واحد أو أكثر من القيم في tuple باستخدام `_` أو `..`، كما سترى في قسم [“Ignoring Values in a Pattern”][ignoring-values-in-a-pattern]<!-- ignore -->. إذا كانت المشكلة هي أن لدينا عددًا كبيرًا جدًا من variables في pattern، فإن الحل هو جعل الأنواع متطابقة عن طريق إزالة variables بحيث يساوي عدد variables عدد العناصر في tuple.

### تعبيرات `if let` الشرطية (if let expressions)

في الفصل 6، ناقشنا كيفية استخدام if let expressions بشكل أساسي كطريقة أقصر لكتابة ما يعادل `match` الذي يطابق حالة واحدة فقط. اختياريًا، يمكن أن تحتوي `if let` على `else` مطابق يحتوي على كود يتم تشغيله إذا لم يطابق pattern في `if let`.

توضح القائمة 19-3 أنه من الممكن أيضًا مزج ومطابقة تعبيرات `if let` و وإلا إذا (else if) و وإلا إذا كان (else if let). يمنحنا القيام بذلك مرونة أكبر من تعبير `match` الذي يمكننا فيه التعبير عن قيمة واحدة فقط للمقارنة مع Patterns. أيضًا، لا يتطلب Rust أن تكون الشروط في سلسلة من أذرع `if let` و `else if` و `else if let` مرتبطة ببعضها البعض.

يحدد الكود في القائمة 19-3 اللون الذي يجب أن يكون عليه الخلفية بناءً على سلسلة من عمليات التحقق لعدة شروط. لهذا المثال، أنشأنا variables بقيم مبرمجة (hardcoded) قد يتلقاها برنامج حقيقي من إدخال المستخدم.

<Listing number="19-3" file-name="src/main.rs" caption="Mixing `if let`, `else if`, `else if let`, and `else`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-03/src/main.rs}}
```

</Listing>

إذا حدد المستخدم لونًا مفضلاً، فسيتم استخدام هذا اللون كخلفية. إذا لم يتم تحديد لون مفضل وكان اليوم هو الثلاثاء، فسيكون لون الخلفية أخضر. بخلاف ذلك، إذا حدد المستخدم عمره كسلسلة نصية وتمكنا من تحليلها كرقم بنجاح، فسيكون اللون إما أرجوانيًا أو برتقاليًا اعتمادًا على قيمة الرقم. إذا لم تنطبق أي من هذه الشروط، فسيكون لون الخلفية أزرق.

تسمح لنا هذه البنية الشرطية بدعم المتطلبات المعقدة. مع القيم المبرمجة التي لدينا هنا، سيطبع هذا المثال `Using purple as the background color`.

يمكنك أن ترى أن `if let` يمكن أن يقدم أيضًا variables جديدة تقوم بـ التظليل (shadowing) لـ variables الموجودة بنفس الطريقة التي يمكن أن تفعلها match arms: السطر `if let Ok(age) = age` يقدم variable `age` جديدًا يحتوي على القيمة داخل متغير `Ok`، مما يقوم بـ shadowing لـ variable `age` الموجود. هذا يعني أننا بحاجة إلى وضع الشرط `if age > 30` داخل تلك الكتلة: لا يمكننا دمج هذين الشرطين في `if let Ok(age) = age && age > 30`. الـ `age` الجديد الذي نريد مقارنته بـ 30 ليس صالحًا حتى يبدأ النطاق (scope) الجديد بالقوس المعقوف.

الجانب السلبي لاستخدام if let expressions هو أن الـ compiler لا يتحقق من الشمولية (exhaustiveness)، بينما يتحقق منها مع تعبيرات `match`. إذا حذفنا كتلة `else` الأخيرة وبالتالي فاتنا التعامل مع بعض الحالات، فلن ينبهنا الـ compiler إلى خطأ المنطق المحتمل.

### حلقة شرطية `while let` (while let conditional loop)

تسمح حلقة شرطية `while let`، المشابهة في البناء لـ `if let`، لحلقة `while` بالاستمرار في العمل طالما استمر pattern في المطابقة. في القائمة 19-4، نعرض حلقة `while let` تنتظر الرسائل المرسلة بين الخيوط (threads)، ولكن في هذه الحالة تتحقق من نتيجة (Result) بدلاً من خيار (Option).

<Listing number="19-4" caption="Using a `while let` loop to print values for as long as `rx.recv()` returns `Ok`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-04/src/main.rs:here}}
```

</Listing>

يطبع هذا المثال `1` و `2` ثم `3`. دالة `recv` (recv method) تأخذ الرسالة الأولى من جانب المستقبل للقناة وتُرجع موافق (`Ok(value)`). عندما رأينا `recv` لأول مرة في الفصل 16، قمنا بفك خطأ (error) مباشرة، أو تفاعلنا معه كمكرر (iterator) باستخدام حلقة `for` (for loop). كما توضح القائمة 19-4، يمكننا أيضًا استخدام `while let`، لأن دالة `recv` تُرجع `Ok` في كل مرة تصل فيها رسالة، طالما أن المرسل موجود، ثم تنتج خطأ (Err) بمجرد قطع اتصال جانب المرسل.

### حلقة `for` (for loop)

في for loop، القيمة التي تتبع الكلمة المفتاحية `for` مباشرة هي pattern. على سبيل المثال، في `for x in y`، فإن `x` هو pattern. توضح القائمة 19-5 كيفية استخدام pattern في for loop لـ destructure، أو تفكيك، tuple كجزء من for loop.

<Listing number="19-5" caption="Using a pattern in a `for` loop to destructure a tuple">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-05/src/main.rs:here}}
```

</Listing>

سيطبع الكود في القائمة 19-5 ما يلي:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-05/output.txt}}
```

نقوم بتكييف iterator باستخدام دالة `enumerate` (enumerate method) بحيث تنتج قيمة وفهرسًا لتلك القيمة، موضوعة في tuple. القيمة الأولى المنتجة هي tuple `(0, 'a')`. عندما تتم مطابقة هذه القيمة مع pattern `(index, value)`، سيكون index هو `0` و value هو `'a'`، مما يطبع السطر الأول من الإخراج.

### معاملات الدالة (Function Parameters)

يمكن أن تكون معاملات الدالة (Function Parameters) أيضًا Patterns. يجب أن يبدو الكود في القائمة 19-6، الذي يعلن عن دالة تسمى `foo` تأخذ معاملًا واحدًا يسمى `x` من النوع `i32`، مألوفًا الآن.

<Listing number="19-6" caption="A function signature using patterns in the parameters">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-06/src/main.rs:here}}
```

</Listing>

الجزء `x` هو pattern! كما فعلنا مع `let`، يمكننا مطابقة tuple في وسائط الدالة بـ pattern. تقسم القائمة 19-7 القيم في tuple أثناء تمريرها إلى دالة.

<Listing number="19-7" file-name="src/main.rs" caption="A function with parameters that destructure a tuple">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-07/src/main.rs}}
```

</Listing>

يطبع هذا الكود `Current location: (3, 5)`. القيم `&(3, 5)` تطابق pattern `&(x, y)`، لذا فإن `x` هي القيمة `3` و `y` هي القيمة `5`.

يمكننا أيضًا استخدام Patterns في قوائم معاملات الإغلاق (closure) بنفس طريقة قوائم Function Parameters لأن closures تشبه الدوال، كما نوقش في الفصل 13.

في هذه المرحلة، رأيت عدة طرق لاستخدام Patterns، ولكن Patterns لا تعمل بنفس الطريقة في كل مكان يمكننا استخدامها فيه. في بعض الأماكن، يجب أن تكون Patterns غير قابلة للدحض (irrefutable)؛ في ظروف أخرى، يمكن أن تكون قابلة للدحض (refutable). سنناقش هذين المفهومين لاحقًا.

[ignoring-values-in-a-pattern]: ch19-03-pattern-syntax.html#ignoring-values-in-a-pattern
