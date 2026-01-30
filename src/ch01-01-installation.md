## التثبيت (Installation)

الخطوة الأولى هي تثبيت Rust. سنقوم بتنزيل Rust من خلال `rustup`، وهي أداة سطر أوامر (command line tool) لإدارة إصدارات Rust والأدوات المرتبطة بها. ستحتاج إلى اتصال بالإنترنت لعملية التنزيل.

> ملاحظة: إذا كنت تفضل عدم استخدام `rustup` لسبب ما، فيرجى الاطلاع على [صفحة طرق تثبيت Rust الأخرى][otherinstall] للحصول على المزيد من الخيارات.

تثبت الخطوات التالية أحدث إصدار مستقر من مترجم لغة Rust (Rust compiler). تضمن ضمانات استقرار Rust أن جميع الأمثلة في الكتاب التي يتم تجميعها (compile) ستستمر في التجميع مع إصدارات Rust الأحدث. قد يختلف الإخراج قليلاً بين الإصدارات لأن Rust غالبًا ما تحسن رسائل الخطأ والتحذيرات. بعبارة أخرى، يجب أن يعمل أي إصدار مستقر أحدث من Rust تقوم بتثبيته باستخدام هذه الخطوات كما هو متوقع مع محتوى هذا الكتاب.

> ### ترميز سطر الأوامر (Command Line Notation)
>
> في هذا الفصل وطوال الكتاب، سنعرض بعض الأوامر المستخدمة في الطرفية (terminal). تبدأ جميع الأسطر التي يجب عليك إدخالها في `terminal` بالرمز `$`. لا تحتاج إلى كتابة الحرف `$`. إنه موجه سطر الأوامر (command line prompt) الذي يظهر للإشارة إلى بداية كل أمر. الأسطر التي لا تبدأ بـ `$ $` عادةً ما تعرض إخراج الأمر السابق. بالإضافة إلى ذلك، ستستخدم الأمثلة الخاصة بـ PowerShell الرمز `>` بدلاً من `$`.

### تثبيت `rustup` على Linux أو macOS

إذا كنت تستخدم Linux أو macOS، فافتح `terminal` وأدخل الأمر التالي:

```console
$ curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
```

يقوم الأمر بتنزيل سكريبت (script) وبدء تثبيت أداة `rustup`، والتي تثبت أحدث إصدار مستقر من Rust. قد يُطلب منك إدخال كلمة المرور الخاصة بك. إذا نجح التثبيت، فسيظهر السطر التالي:

```text
Rust is installed now. Great!
```

ستحتاج أيضًا إلى رابط (linker)، وهو برنامج تستخدمه Rust لضم مخرجاتها المجمعة في ملف واحد. من المحتمل أن يكون لديك واحد بالفعل. إذا تلقيت أخطاء `linker`، فيجب عليك تثبيت مترجم لغة C (C compiler)، والذي سيتضمن عادةً `linker`. يعد `C compiler` مفيدًا أيضًا لأن بعض حزم Rust الشائعة تعتمد على كود C وستحتاج إلى `C compiler`.

على macOS، يمكنك الحصول على `C compiler` عن طريق تشغيل:

```console
$ xcode-select --install
```

يجب على مستخدمي Linux عمومًا تثبيت GCC أو Clang، وفقًا لوثائق التوزيعة (distribution) الخاصة بهم. على سبيل المثال، إذا كنت تستخدم Ubuntu، يمكنك تثبيت حزمة `build-essential`.

### تثبيت `rustup` على Windows

على Windows، انتقل إلى [https://www.rust-lang.org/tools/install][install] واتبع التعليمات لتثبيت Rust. في مرحلة ما من التثبيت، سيُطلب منك تثبيت Visual Studio. يوفر هذا `linker` والمكتبات الأصلية (native libraries) اللازمة لتجميع البرامج. إذا كنت بحاجة إلى مزيد من المساعدة في هذه الخطوة، فراجع [https://rust-lang.github.io/rustup/installation/windows-msvc.html][msvc].

يستخدم باقي هذا الكتاب أوامر تعمل في كل من _cmd.exe_ و PowerShell. إذا كانت هناك اختلافات محددة، فسنشرح أيها يجب استخدامه.

### استكشاف الأخطاء وإصلاحها (Troubleshooting)

للتحقق مما إذا كنت قد قمت بتثبيت Rust بشكل صحيح، افتح `shell` وأدخل هذا السطر:

```console
$ rustc --version
```

يجب أن ترى رقم الإصدار (version number)، ورمز التثبيت (commit hash)، وتاريخ التثبيت (commit date) لأحدث إصدار مستقر تم إصداره، بالتنسيق التالي:

```text
rustc x.y.z (abcabcabc yyyy-mm-dd)
```

إذا رأيت هذه المعلومات، فقد قمت بتثبيت Rust بنجاح! إذا لم تر هذه المعلومات، فتحقق من أن Rust موجود في متغير النظام `%PATH%` الخاص بك على النحو التالي.

في Windows CMD، استخدم:

```console
> echo %PATH%
```

في PowerShell، استخدم:

```powershell
> echo $env:Path
```

في Linux و macOS، استخدم:

```console
$ echo $PATH
```

إذا كان كل هذا صحيحًا ولا تزال Rust لا تعمل، فهناك عدد من الأماكن التي يمكنك الحصول على المساعدة منها. اكتشف كيفية التواصل مع Rustaceans الآخرين (لقب سخيف نطلق به على أنفسنا) على [صفحة المجتمع][community].

### التحديث وإلغاء التثبيت (Updating and Uninstalling)

بمجرد تثبيت Rust عبر `rustup`، يصبح التحديث إلى إصدار تم إصداره حديثًا أمرًا سهلاً. من `shell` الخاص بك، قم بتشغيل سكريبت التحديث التالي:

```console
$ rustup update
```

لإلغاء تثبيت Rust و `rustup`، قم بتشغيل سكريبت إلغاء التثبيت التالي من `shell` الخاص بك:

```console
$ rustup self uninstall
```

<!-- Old headings. Do not remove or links may break. -->
<a id="local-documentation"></a>

### قراءة الوثائق المحلية (Reading the Local Documentation)

يتضمن تثبيت Rust أيضًا نسخة محلية من الوثائق حتى تتمكن من قراءتها دون اتصال بالإنترنت. قم بتشغيل `rustup doc` لفتح الوثائق المحلية في متصفحك.

في أي وقت يتم فيه توفير نوع (type) أو دالة (function) بواسطة المكتبة القياسية (standard library) ولست متأكدًا مما تفعله أو كيفية استخدامها، استخدم وثائق واجهة برمجة التطبيقات (API) لمعرفة ذلك!

<!-- Old headings. Do not remove or links may break. -->
<a id="text-editors-and-integrated-development-environments"></a>

### استخدام محررات النصوص وبيئات التطوير المتكاملة (Using Text Editors and IDEs)

لا يفترض هذا الكتاب أي شيء حول الأدوات التي تستخدمها لكتابة كود Rust. أي محرر نصوص تقريبًا سيقوم بالمهمة! ومع ذلك، فإن العديد من محررات النصوص و `IDEs` لديها دعم مدمج لـ Rust. يمكنك دائمًا العثور على قائمة حديثة إلى حد ما للعديد من المحررات و `IDEs` على [صفحة الأدوات][tools] على موقع Rust على الويب.

### العمل دون اتصال بالإنترنت مع هذا الكتاب (Working Offline with This Book)

في العديد من الأمثلة، سنستخدم حزم Rust تتجاوز `standard library`. للعمل من خلال تلك الأمثلة، ستحتاج إما إلى اتصال بالإنترنت أو إلى تنزيل تلك التبعيات (dependencies) مسبقًا. لتنزيل `dependencies` مسبقًا، يمكنك تشغيل الأوامر التالية. (سنشرح ما هو `cargo` وما يفعله كل من هذه الأوامر بالتفصيل لاحقًا.)

```console
$ cargo new get-dependencies
$ cd get-dependencies
$ cargo add rand@0.8.5 trpl@0.2.0
```

سيؤدي هذا إلى تخزين التنزيلات لهذه الحزم مؤقتًا (cache) بحيث لن تحتاج إلى تنزيلها لاحقًا. بمجرد تشغيل هذا الأمر، لا تحتاج إلى الاحتفاظ بمجلد `get-dependencies`. إذا قمت بتشغيل هذا الأمر، يمكنك استخدام العلامة `--offline` مع جميع أوامر `cargo` في بقية الكتاب لاستخدام هذه الإصدارات المخزنة مؤقتًا بدلاً من محاولة استخدام الشبكة.

[otherinstall]: https://forge.rust-lang.org/infra/other-installation-methods.html
[install]: https://www.rust-lang.org/tools/install
[msvc]: https://rust-lang.github.io/rustup/installation/windows-msvc.html
[community]: https://www.rust-lang.org/community
[tools]: https://www.rust-lang.org/tools
