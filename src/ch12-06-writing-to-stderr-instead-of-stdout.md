<!-- Old headings. Do not remove or links may break. -->

<a id="writing-error-messages-to-standard-error-instead-of-standard-output"></a>

## إعادة توجيه الأخطاء إلى الخطأ القياسي (Redirecting Errors to Standard Error)

في الوقت الحالي، نقوم بكتابة جميع مخرجاتنا إلى الطرفية (terminal) باستخدام ماكرو (macro) `println!`. في معظم الطرفيات، هناك نوعان من المخرجات: _المخرجات القياسية_ (standard output) وتختصر بـ (`stdout`) للمعلومات العامة، و _الخطأ القياسي_ (standard error) ويختصر بـ (`stderr`) لرسائل الخطأ. يتيح هذا التمييز للمستخدمين اختيار توجيه المخرجات الناجحة لبرنامج ما إلى ملف مع الاستمرار في طباعة رسائل الخطأ على الشاشة.

ماكرو `println!` قادر فقط على الطباعة إلى standard output، لذا يتعين علينا استخدام شيء آخر للطباعة إلى standard error.

### التحقق من مكان كتابة الأخطاء (Checking Where Errors Are Written)

أولاً، دعونا نلاحظ كيف يتم حالياً كتابة المحتوى المطبوع بواسطة `minigrep` إلى standard output، بما في ذلك أي رسائل خطأ نريد كتابتها إلى standard error بدلاً من ذلك. سنفعل ذلك عن طريق إعادة توجيه تدفق standard output إلى ملف مع التسبب في حدوث خطأ عمداً. لن نقوم بإعادة توجيه تدفق standard error، لذا فإن أي محتوى يتم إرساله إلى standard error سيستمر في الظهور على الشاشة.

يُتوقع من برامج واجهة الأوامر (Command line programs) إرسال رسائل الخطأ إلى تدفق standard error حتى نتمكن من رؤية رسائل الخطأ على الشاشة حتى لو قمنا بإعادة توجيه تدفق standard output إلى ملف. برنامجنا حالياً لا يتصرف بشكل جيد: نحن على وشك أن نرى أنه يحفظ مخرجات رسالة الخطأ في ملف بدلاً من ذلك!

لإظهار هذا السلوك، سنقوم بتشغيل البرنامج باستخدام `>` ومسار الملف، _output.txt_، الذي نريد إعادة توجيه تدفق standard output إليه. لن نمرر أي وسائط (arguments)، مما قد يتسبب في حدوث خطأ:

```console
$ cargo run > output.txt
```

تخبر صيغة `>` الغلاف (shell) بكتابة محتويات standard output إلى _output.txt_ بدلاً من الشاشة. لم نرَ رسالة الخطأ التي كنا نتوقع طباعتها على الشاشة، وهذا يعني أنها لا بد وأن انتهت في الملف. وهذا ما يحتويه _output.txt_:

```text
Problem parsing arguments: not enough arguments
```

نعم، تتم طباعة رسالة الخطأ الخاصة بنا إلى standard output. من المفيد أكثر بكثير طباعة رسائل خطأ كهذه إلى standard error بحيث تنتهي البيانات الناتجة عن تشغيل ناجح فقط في الملف. سنقوم بتغيير ذلك.

### طباعة الأخطاء إلى الخطأ القياسي (Printing Errors to Standard Error)

سنستخدم الكود الموجود في القائمة 12-24 لتغيير كيفية طباعة رسائل الخطأ. بسبب إعادة الهيكلة (refactoring) التي قمنا بها سابقاً في هذا الفصل، فإن كل الكود الذي يطبع رسائل الخطأ موجود في دالة واحدة، وهي `main`. توفر المكتبة القياسية (standard library) ماكرو `eprintln!` الذي يطبع إلى تدفق standard error، لذا دعونا نغير المكانين اللذين كنا نستدعي فيهما `println!` لطباعة الأخطاء لاستخدام `eprintln!` بدلاً من ذلك.

<Listing number="12-24" file-name="src/main.rs" caption="كتابة رسائل الخطأ إلى الخطأ القياسي بدلاً من المخرجات القياسية باستخدام `eprintln!`">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-24/src/main.rs:here}}
```

</Listing>

دعونا الآن نشغل البرنامج مرة أخرى بنفس الطريقة، بدون أي arguments ومع إعادة توجيه standard output باستخدام `>`:

```console
$ cargo run > output.txt
Problem parsing arguments: not enough arguments
```

الآن نرى الخطأ على الشاشة ولا يحتوي _output.txt_ على شيء، وهو السلوك الذي نتوقعه من Command line programs.

دعونا نشغل البرنامج مرة أخرى مع arguments لا تسبب خطأ ولكن مع الاستمرار في إعادة توجيه standard output إلى ملف، هكذا:

```console
$ cargo run -- to poem.txt > output.txt
```

لن نرى أي مخرجات في terminal، وسوف يحتوي _output.txt_ على نتائجنا:

<span class="filename">اسم الملف: output.txt</span>

```text
Are you nobody, too?
How dreary to be somebody!
```

يوضح هذا أننا نستخدم الآن standard output للمخرجات الناجحة و standard error لمخرجات الخطأ حسب الاقتضاء.

## ملخص (Summary)

لخص هذا الفصل بعض المفاهيم الرئيسية التي تعلمتها حتى الآن وغطى كيفية إجراء عمليات الإدخال والإخراج (I/O operations) الشائعة في Rust. باستخدام arguments واجهة الأوامر، والملفات، ومتغيرات البيئة (environment variables)، وماكرو `eprintln!` لطباعة الأخطاء، فأنت الآن مستعد لكتابة تطبيقات واجهة الأوامر. وبالجمع مع المفاهيم الواردة في الفصول السابقة، سيكون الكود الخاص بك منظماً جيداً، ويخزن البيانات بفعالية في هياكل البيانات (data structures) المناسبة، ويعالج الأخطاء بشكل جيد، ويكون مختبراً جيداً.

بعد ذلك، سنستكشف بعض ميزات Rust التي تأثرت باللغات الوظيفية (functional languages): الإغلاقات (closures) والمكررات (iterators).
