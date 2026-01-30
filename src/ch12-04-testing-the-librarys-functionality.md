<!-- Old headings. Do not remove or links may break. -->
<a id="developing-the-librarys-functionality-with-test-driven-development"></a>

## إضافة الوظائف باستخدام التطوير القائم على الاختبار (Adding Functionality with Test-Driven Development)

الآن بعد أن أصبح لدينا منطق البحث في _src/lib.rs_ منفصلاً عن دالة `main` ، أصبح من الأسهل بكثير كتابة اختبارات (tests) للوظائف الأساسية لكودنا. يمكننا استدعاء الدوال مباشرة بـ معاملات (arguments) متنوعة والتحقق من قيم الإرجاع (return values) دون الحاجة إلى استدعاء ملفنا الثنائي (binary) من سطر الأوامر.

في هذا القسم، سنضيف منطق البحث إلى برنامج `minigrep` باستخدام عملية التطوير القائم على الاختبار (test-driven development - TDD) بالخطوات التالية:

1. اكتب اختباراً يفشل وقم بتشغيله للتأكد من فشله للسبب الذي تتوقعه.
2. اكتب أو عدل ما يكفي من الكود فقط لجعل الاختبار الجديد ينجح.
3. قم بإعادة هيكلة (Refactor) الكود الذي أضفته أو غيرته للتو وتأكد من استمرار نجاح الاختبارات.
4. كرر من الخطوة 1!

على الرغم من أنها مجرد واحدة من طرق عديدة لكتابة البرمجيات، إلا أن TDD يمكن أن يساعد في دفع تصميم الكود. كتابة الاختبار قبل كتابة الكود الذي يجعل الاختبار ينجح يساعد في الحفاظ على تغطية اختبار (test coverage) عالية طوال العملية.

سنقوم بتطوير الوظيفة التي ستقوم فعلياً بالبحث عن سلسلة الاستعلام (query string) في محتويات الملف وإنتاج قائمة بالأسطر التي تطابق الاستعلام. سنضيف هذه الوظيفة في دالة تسمى `search`.

### كتابة اختبار فاشل (Writing a Failing Test)

في _src/lib.rs_ ، سنضيف وحدة اختبارات (tests module) مع دالة اختبار، كما فعلنا في [الفصل الحادي عشر][ch11-anatomy]. تحدد دالة الاختبار السلوك الذي نريده لدالة `search`: ستأخذ استعلاماً (query) والنص المراد البحث فيه، وستعيد فقط الأسطر من النص التي تحتوي على الاستعلام. توضح القائمة 12-15 هذا الاختبار.

<Listing number="12-15" file-name="src/lib.rs" caption="Creating a failing test for the `search` function for the functionality we wish we had">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-15/src/lib.rs:here}}
```

</Listing>

يبحث هذا الاختبار عن السلسلة `"duct"`. النص الذي نبحث فيه يتكون من ثلاثة أسطر، واحد منها فقط يحتوي على `"duct"` (لاحظ أن الشرطة المائلة العكسية بعد علامة الاقتباس المزدوجة الافتتاحية تخبر Rust بعدم وضع حرف سطر جديد في بداية محتويات سلسلة النص هذه). نحن نؤكد (assert) أن القيمة المعادة من دالة `search` تحتوي فقط على السطر الذي نتوقعه.

إذا قمنا بتشغيل هذا الاختبار، فسيفشل حالياً لأن ماكرو `unimplemented!` يسبب حالة ذعر (panics) مع الرسالة "not implemented". وفقاً لمبادئ TDD، سنتخذ خطوة صغيرة بإضافة ما يكفي من الكود فقط لكي لا تسبب الدالة panic عند استدعائها عن طريق تعريف دالة `search` لتعيد دائماً متجهاً (vector) فارغاً، كما هو موضح في القائمة 12-16. بعد ذلك، يجب أن يتم تجميع (compile) الاختبار ويفشل لأن المتجه الفارغ لا يطابق متجراً يحتوي على السطر `"safe, fast, productive."`.

<Listing number="12-16" file-name="src/lib.rs" caption="Defining just enough of the `search` function so that calling it won’t panic">

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-16/src/lib.rs:here}}
```

</Listing>

الآن دعونا نناقش سبب حاجتنا إلى تعريف عمر (lifetime) صريح `'a` في توقيع (signature) دالة `search` واستخدام ذلك الـ lifetime مع argument الـ `contents` وقيمة الإرجاع. تذكر في [الفصل العاشر][ch10-lifetimes] أن معاملات العمر (lifetime parameters) تحدد أي عمر للمعاملات مرتبط بعمر قيمة الإرجاع. في هذه الحالة، نشير إلى أن الـ vector المعاد يجب أن يحتوي على شرائح نصية (string slices) تشير إلى شرائح من الـ argument الـ `contents` (بدلاً من الـ argument الـ `query`).

بعبارة أخرى، نحن نخبر Rust أن البيانات المعادة بواسطة دالة `search` ستعيش طالما عاشت البيانات الممرة إلى دالة `search` في الـ argument الـ `contents`. هذا أمر مهم! البيانات المشار إليها _بواسطة_ شريحة (slice) يجب أن تكون صالحة لكي يكون المرجع (reference) صالحاً؛ إذا افترض المترجم (compiler) أننا ننشئ string slices من `query` بدلاً من `contents` ، فسيقوم بفحص الأمان بشكل غير صحيح.

إذا نسينا تعليقات العمر (lifetime annotations) وحاولنا تجميع هذه الدالة، فسنحصل على هذا الخطأ:

```console
{{#include ../listings/ch12-an-io-project/output-only-02-missing-lifetimes/output.txt}}
```

لا يمكن لـ Rust معرفة أي من المعاملين نحتاجه للمخرجات، لذا نحتاج إلى إخباره صراحة. لاحظ أن نص المساعدة يقترح تحديد نفس lifetime parameter لجميع المعاملات ونوع المخرجات، وهذا غير صحيح! لأن `contents` هو المعامل الذي يحتوي على كل النص الخاص بنا ونريد إعادة أجزاء من ذلك النص التي تتطابق، فنحن نعلم أن `contents` هو المعامل الوحيد الذي يجب أن يكون مرتبطاً بقيمة الإرجاع باستخدام صيغة lifetime.

لا تتطلب لغات البرمجة الأخرى منك ربط المعاملات بقيم الإرجاع في الـ signature، ولكن هذه الممارسة ستصبح أسهل بمرور الوقت. قد ترغب في مقارنة هذا المثال بالأمثلة الموجودة في قسم ["التحقق من المراجع باستخدام الأعمار"][validating-references-with-lifetimes] في الفصل العاشر.

### كتابة الكود لإنجاح الاختبار (Writing Code to Pass the Test)

حالياً، يفشل اختبارنا لأننا نعيد دائماً vector فارغاً. لإصلاح ذلك وتنفيذ `search` ، يحتاج برنامجنا إلى اتباع هذه الخطوات:

1. التكرار (Iterate) عبر كل سطر من المحتويات.
2. التحقق مما إذا كان السطر يحتوي على سلسلة الاستعلام الخاصة بنا.
3. إذا كان يحتوي عليها، أضفه إلى قائمة القيم التي نعيدها.
4. إذا لم يكن يحتوي عليها، فلا تفعل شيئاً.
5. أعد قائمة النتائج المتطابقة.

دعونا نعمل على كل خطوة، بدءاً من التكرار عبر الأسطر.

#### التكرار عبر الأسطر باستخدام طريقة `lines` (Iterating Through Lines with the `lines` Method)

تمتلك Rust طريقة مفيدة للتعامل مع التكرار سطراً بسطر للسلاسل النصية، تسمى بشكل ملائم `lines` ، والتي تعمل كما هو موضح في القائمة 12-17. لاحظ أن هذا لن يتم تجميعه بعد.

<Listing number="12-17" file-name="src/lib.rs" caption="Iterating through each line in `contents`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-17/src/lib.rs:here}}
```

</Listing>

تعيد طريقة `lines` مكرراً (iterator). سنتحدث عن الـ iterators بعمق في [الفصل الثالث عشر][ch13-iterators]. لكن تذكر أنك رأيت هذه الطريقة لاستخدام iterator في [القائمة 3-5][ch3-iter] ، حيث استخدمنا حلقة `for` مع iterator لتشغيل بعض الكود على كل عنصر في مجموعة (collection).

#### البحث في كل سطر عن الاستعلام (Searching Each Line for the Query)

بعد ذلك، سنتحقق مما إذا كان السطر الحالي يحتوي على سلسلة الاستعلام الخاصة بنا. لحسن الحظ، تمتلك السلاسل النصية طريقة مفيدة تسمى `contains` تقوم بذلك نيابة عنا! أضف استدعاءً لطريقة `contains` في دالة `search` ، كما هو موضح في القائمة 12-18. لاحظ أن هذا لا يزال لن يتم تجميعه بعد.

<Listing number="12-18" file-name="src/lib.rs" caption="Adding functionality to see whether the line contains the string in `query`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-18/src/lib.rs:here}}
```

</Listing>

في الوقت الحالي، نحن نبني الوظائف. لكي يتم تجميع الكود، نحتاج إلى إعادة قيمة من الجسم كما أشرنا في signature الدالة.

#### تخزين الأسطر المتطابقة (Storing Matching Lines)

لإنهاء هذه الدالة، نحتاج إلى طريقة لتخزين الأسطر المتطابقة التي نريد إعادتها. لذلك، يمكننا إنشاء vector قابل للتغيير (mutable vector) قبل حلقة `for` واستدعاء طريقة `push` لتخزين `line` في الـ vector. بعد حلقة `for` ، نعيد الـ vector، كما هو موضح في القائمة 12-19.

<Listing number="12-19" file-name="src/lib.rs" caption="Storing the lines that match so that we can return them">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-19/src/lib.rs:here}}
```

</Listing>

الآن يجب أن تعيد دالة `search` فقط الأسطر التي تحتوي على `query` ، ويجب أن ينجح اختبارنا. دعونا نشغل الاختبار:

```console
{{#include ../listings/ch12-an-io-project/listing-12-19/output.txt}}
```

لقد نجح اختبارنا، لذا نحن نعلم أنه يعمل!

في هذه المرحلة، يمكننا التفكير في فرص لإعادة هيكلة تنفيذ دالة البحث مع الحفاظ على نجاح الاختبارات للحفاظ على نفس الوظيفة. الكود في دالة البحث ليس سيئاً للغاية، لكنه لا يستفيد من بعض الميزات المفيدة للـ iterators. سنعود إلى هذا المثال في [الفصل الثالث عشر][ch13-iterators] ، حيث سنستكشف الـ iterators بالتفصيل، وننظر في كيفية تحسينه.

الآن يجب أن يعمل البرنامج بأكمله! دعونا نجربه، أولاً بكلمة يجب أن تعيد سطراً واحداً بالضبط من قصيدة إميلي ديكنسون: _frog_.

```console
{{#include ../listings/ch12-an-io-project/no-listing-02-using-search-in-run/output.txt}}
```

رائع! الآن دعونا نجرب كلمة ستطابق أسطراً متعددة، مثل _body_:

```console
{{#include ../listings/ch12-an-io-project/output-only-03-multiple-matches/output.txt}}
```

وأخيراً، دعونا نتأكد من أننا لا نحصل على أي أسطر عندما نبحث عن كلمة ليست موجودة في أي مكان في القصيدة، مثل _monomorphization_:

```console
{{#include ../listings/ch12-an-io-project/output-only-04-no-matches/output.txt}}
```

ممتاز! لقد بنينا نسختنا المصغرة الخاصة من أداة كلاسيكية وتعلمنا الكثير عن كيفية هيكلة التطبيقات. لقد تعلمنا أيضاً القليل عن إدخال وإخراج الملفات، والأعمار، والاختبار، وتحليل سطر الأوامر.

لإكمال هذا المشروع، سنوضح بإيجاز كيفية التعامل مع متغيرات البيئة (environment variables) وكيفية الطباعة إلى الخطأ القياسي (standard error)، وكلاهما مفيد عندما تكتب برامج سطر الأوامر.

[validating-references-with-lifetimes]: ch10-03-lifetime-syntax.html#validating-references-with-lifetimes
[ch11-anatomy]: ch11-01-writing-tests.html#the-anatomy-of-a-test-function
[ch10-lifetimes]: ch10-03-lifetime-syntax.html
[ch3-iter]: ch03-05-control-flow.html#looping-through-a-collection-with-for
[ch13-iterators]: ch13-02-iterators.html
