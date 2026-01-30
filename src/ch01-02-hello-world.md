## مرحباً أيها العالم! (Hello, World!)

الآن بعد أن قمت بتثبيت Rust، حان الوقت لكتابة أول برنامج Rust لك. من التقليدي عند تعلم لغة جديدة كتابة برنامج صغير يطبع النص `Hello, world!` على الشاشة، لذلك سنفعل الشيء نفسه هنا!

> ملاحظة: يفترض هذا الكتاب إلمامًا أساسيًا بسطر الأوامر (command line). لا تفرض Rust متطلبات محددة حول التحرير أو الأدوات الخاصة بك أو مكان وجود الكود الخاص بك، لذلك إذا كنت تفضل استخدام بيئة التطوير المتكاملة (IDE) بدلاً من `command line`، فلا تتردد في استخدام `IDE` المفضل لديك. تحتوي العديد من `IDEs` الآن على درجة معينة من دعم Rust؛ تحقق من وثائق `IDE` للحصول على التفاصيل. يركز فريق Rust على تمكين دعم رائع لـ `IDE` عبر محلل لغة Rust (rust-analyzer). راجع [الملحق د][devtools] لمزيد من التفاصيل.

<!-- Old headings. Do not remove or links may break. -->
<a id="creating-a-project-directory"></a>

### إعداد دليل المشروع (Project Directory Setup)

ستبدأ بإنشاء دليل (directory) لتخزين كود Rust الخاص بك. لا يهم Rust مكان وجود الكود الخاص بك، ولكن بالنسبة للتمارين والمشاريع في هذا الكتاب، نقترح إنشاء دليل المشاريع (projects directory) في دليلك الرئيسي والاحتفاظ بجميع مشاريعك هناك.

افتح الطرفية (terminal) وأدخل الأوامر التالية لإنشاء `projects directory` ودليل لمشروع "Hello, world!" داخل `projects directory`.

بالنسبة لنظامي Linux و macOS و PowerShell على Windows، أدخل ما يلي:

```console
$ mkdir ~/projects
$ cd ~/projects
$ mkdir hello_world
$ cd hello_world
```

بالنسبة لـ Windows CMD، أدخل ما يلي:

```cmd
> mkdir "%USERPROFILE%\projects"
> cd /d "%USERPROFILE%\projects"
> mkdir hello_world
> cd hello_world
```

<!-- Old headings. Do not remove or links may break. -->
<a id="writing-and-running-a-rust-program"></a>

### أساسيات برنامج Rust (Rust Program Basics)

بعد ذلك، قم بإنشاء ملف مصدر (source file) جديد وسمه _main.rs_. تنتهي ملفات Rust دائمًا بامتداد `.rs` (extension). إذا كنت تستخدم أكثر من كلمة في اسم ملفك، فإن الاصطلاح (convention) هو استخدام شرطة سفلية (underscore) للفصل بينها. على سبيل المثال، استخدم _hello_world.rs_ بدلاً من _helloworld.rs_.

الآن افتح ملف _main.rs_ الذي أنشأته للتو وأدخل الكود في القائمة 1-1.

<Listing number="1-1" file-name="main.rs" caption="برنامج يطبع `Hello, world!`">

```rust
fn main() {
    println!("Hello, world!");
}
```

</Listing>

احفظ الملف وعد إلى نافذة `terminal` الخاصة بك في دليل _~/projects/hello_world_. على Linux أو macOS، أدخل الأوامر التالية لتجميع (compile) وتشغيل (run) الملف:

```console
$ rustc main.rs
$ ./main
Hello, world!
```

على Windows، أدخل الأمر `.\main` بدلاً من `./main`:

```powershell
> rustc main.rs
> .\main
Hello, world!
```

بغض النظر عن نظام التشغيل (operating system) الخاص بك، يجب أن تتم طباعة السلسلة النصية (string) `Hello, world!` على `terminal`. إذا لم تر هذا الإخراج، فارجع إلى جزء [“استكشاف الأخطاء وإصلاحها”][troubleshooting] من قسم التثبيت للحصول على طرق للمساعدة.

إذا تمت طباعة `Hello, world!`، فتهانينا! لقد كتبت رسميًا برنامج Rust. هذا يجعلك مبرمج Rust - مرحبًا بك!

<!-- Old headings. Do not remove or links may break. -->

<a id="anatomy-of-a-rust-program"></a>

### تشريح برنامج Rust (The Anatomy of a Rust Program)

دعنا نراجع برنامج "Hello, world!" هذا بالتفصيل. إليك الجزء الأول من اللغز:

```rust
fn main() {

}
```

تحدد هذه الأسطر دالة (function) تسمى `main`. الدالة الرئيسية (`main function`) خاصة: إنها دائمًا أول كود يتم تشغيله في كل برنامج Rust قابل للتنفيذ (executable). هنا، يعلن السطر الأول عن `function` تسمى `main` ليس لها معاملات (parameters) ولا تُرجع شيئًا. إذا كانت هناك `parameters`، فستكون داخل الأقواس (`()`).

جسم الدالة (function body) مغلف بأقواس معقوفة (curly brackets) `{}`. تتطلب Rust `curly brackets` حول جميع أجسام الدوال. من الأسلوب الجيد وضع القوس المعقوف الافتتاحي في نفس السطر مثل إعلان `function`، مع إضافة مسافة واحدة بينهما.

> ملاحظة: إذا كنت ترغب في الالتزام بأسلوب قياسي عبر مشاريع Rust، يمكنك استخدام أداة منسق الكود (rustfmt) التلقائية لتنسيق الكود الخاص بك بأسلوب معين (المزيد حول `rustfmt` في [الملحق د][devtools]). قام فريق Rust بتضمين هذه الأداة مع توزيع Rust القياسي، مثل `rustc`، لذلك يجب أن تكون مثبتة بالفعل على جهاز الكمبيوتر الخاص بك!

يحتوي `function body` لـ `main` على الكود التالي:

```rust
println!("Hello, world!");
```

يقوم هذا السطر بكل العمل في هذا البرنامج الصغير: إنه يطبع نصًا على الشاشة. هناك ثلاث تفاصيل مهمة يجب ملاحظتها هنا.

أولاً، `println!` يستدعي ماكرو (macro) Rust. إذا كان قد استدعى `function` بدلاً من ذلك، فسيتم إدخاله كـ `println` (بدون `!`). `macros` Rust هي طريقة لكتابة كود يولد كودًا لتوسيع بناء جملة Rust، وسنناقشها بمزيد من التفصيل في [الفصل 20][ch20-macros]. في الوقت الحالي، تحتاج فقط إلى معرفة أن استخدام `!` يعني أنك تستدعي `macro` بدلاً من `function` عادية وأن `macros` لا تتبع دائمًا نفس قواعد `functions`.

ثانيًا، ترى `string` `"Hello, world!"`. نمرر هذه `string` كوسيط (argument) إلى `println!`، ويتم طباعة `string` على الشاشة.

ثالثًا، ننهي السطر بفاصلة منقوطة (semicolon) (`;`)، مما يشير إلى أن هذا التعبير (expression) قد انتهى، والتعبير التالي جاهز للبدء. تنتهي معظم أسطر كود Rust بـ `semicolon`.

<!-- Old headings. Do not remove or links may break. -->
<a id="compiling-and-running-are-separate-steps"></a>

### التجميع والتنفيذ (Compilation and Execution)

لقد قمت للتو بتشغيل برنامج تم إنشاؤه حديثًا، لذا دعنا نفحص كل خطوة في العملية.

قبل تشغيل برنامج Rust، يجب عليك تجميعه باستخدام مترجم Rust (Rust compiler) عن طريق إدخال الأمر `rustc` وتمرير اسم ملف المصدر (source file) الخاص بك إليه، مثل هذا:

```console
$ rustc main.rs
```

إذا كانت لديك خلفية في C أو C++، فستلاحظ أن هذا مشابه لـ `gcc` أو `clang`. بعد التجميع بنجاح، يخرج Rust ملف ثنائي قابل للتنفيذ (binary executable).

على Linux و macOS و PowerShell على Windows، يمكنك رؤية `executable` عن طريق إدخال الأمر `ls` في `shell` الخاص بك:

```console
$ ls
main  main.rs
```

على Linux و macOS، سترى ملفين. باستخدام PowerShell على Windows، سترى نفس الملفات الثلاثة التي تراها باستخدام CMD. باستخدام CMD على Windows، ستدخل ما يلي:

```cmd
> dir /B %= the /B option says to only show the file names =%
main.exe
main.pdb
main.rs
```

يعرض هذا `source file` بامتداد `.rs`، و `executable` (_main.exe_ على Windows، ولكن _main_ على جميع `operating systems` الأخرى)، وعند استخدام Windows، ملف يحتوي على معلومات تصحيح الأخطاء (debugging information) بامتداد _.pdb_. من هنا، تقوم بتشغيل ملف _main_ أو _main.exe_، مثل هذا:

```console
$ ./main # or .\main on Windows
```

إذا كان _main.rs_ هو برنامج "Hello, world!" الخاص بك، فإن هذا السطر يطبع `Hello, world!` على `terminal` الخاص بك.

إذا كنت أكثر دراية بلغة ديناميكية (dynamic language)، مثل Ruby أو Python أو JavaScript، فقد لا تكون معتادًا على تجميع وتشغيل برنامج كخطوات منفصلة. Rust هي لغة مجمعة مسبقاً (ahead-of-time compiled language)، مما يعني أنه يمكنك تجميع برنامج وإعطاء `executable` لشخص آخر، ويمكنه تشغيله حتى بدون تثبيت Rust. إذا أعطيت شخصًا ملف _.rb_ أو _.py_ أو _.js_، فإنه يحتاج إلى تثبيت تطبيق Ruby أو Python أو JavaScript (على التوالي). ولكن في تلك اللغات، تحتاج فقط إلى أمر واحد لتجميع وتشغيل برنامجك. كل شيء هو مقايضة في تصميم اللغة.

التجميع باستخدام `rustc` فقط جيد للبرامج البسيطة، ولكن مع نمو مشروعك، سترغب في إدارة جميع الخيارات وتسهيل مشاركة الكود الخاص بك. بعد ذلك، سنقدم لك أداة كارغو (Cargo)، والتي ستساعدك في كتابة برامج Rust واقعية.

[troubleshooting]: ch01-01-installation.html#troubleshooting
[devtools]: appendix-04-useful-development-tools.html
[ch20-macros]: ch20-05-macros.html
