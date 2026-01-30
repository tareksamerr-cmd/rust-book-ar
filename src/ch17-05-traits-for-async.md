<!-- Old headings. Do not remove or links may break. -->

<a id="digging-into-the-traits-for-async"></a>

## نظرة فاحصة على سمات البرمجة غير المتزامنة (Traits for Async)

خلال هذا الفصل، استخدمنا سمات (traits) مثل `Future` و `Stream` و `StreamExt` بطرق متنوعة. ومع ذلك، فقد تجنبنا حتى الآن الخوض في تفاصيل كيفية عملها أو كيفية ترابطها معاً، وهو أمر جيد في معظم الأوقات لعملك اليومي بلغة Rust. لكن في بعض الأحيان، ستواجه مواقف تحتاج فيها إلى فهم المزيد من تفاصيل هذه الـ traits، جنباً إلى جنب مع نوع `Pin` وسمة `Unpin`. في هذا القسم، سنتعمق بما يكفي للمساعدة في تلك السيناريوهات، مع ترك التعمق _الحقيقي_ للتوثيقات الأخرى.

<!-- Old headings. Do not remove or links may break. -->

<a id="future"></a>

### سمة `Future`

لنبدأ بإلقاء نظرة فاحصة على كيفية عمل سمة `Future`. إليك كيف تعرفها لغة Rust:

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

pub trait Future {
    type Output;

    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

يتضمن تعريف الـ trait هذا مجموعة من الأنواع الجديدة وأيضاً بعض الصيغ (syntax) التي لم نرها من قبل، لذا دعونا نستعرض التعريف جزءاً بجزء.

أولاً، يحدد النوع المرتبط (associated type) لـ `Future` وهو `Output` ما ستؤول إليه الـ future عند اكتمالها. هذا مشابه للنوع المرتبط `Item` في سمة `Iterator`.
ثانياً، تمتلك `Future` طريقة (method) تسمى `poll` ، والتي تأخذ مرجع (reference) خاصاً من نوع `Pin` لمعامل `self` الخاص بها، ومرجعاً قابلاً للتغيير (mutable reference) لنوع `Context` ، وتعيد `Poll<Self::Output>`. سنتحدث أكثر عن `Pin` و `Context` بعد قليل. في الوقت الحالي، دعونا نركز على ما تعيده الـ method، وهو نوع `Poll`:

```rust
pub enum Poll<T> {
    Ready(T),
    Pending,
}
```

نوع `Poll` هذا مشابه لـ `Option`. لديه متغير (variant) واحد يحتوي على قيمة، وهو `Ready(T)` ، وآخر لا يحتوي على قيمة، وهو `Pending`. ومع ذلك، فإن `Poll` يعني شيئاً مختلفاً تماماً عن `Option`! يشير الـ variant المسمى `Pending` إلى أن الـ future لا تزال لديها أعمال للقيام بها، لذا سيحتاج المستدعي (caller) إلى التحقق مرة أخرى لاحقاً. بينما يشير الـ variant المسمى `Ready` إلى أن الـ `Future` قد أنهت عملها وأن القيمة `T` أصبحت متاحة.

> ملاحظة: من النادر أن تحتاج إلى استدعاء `poll` مباشرة، ولكن إذا اضطررت لذلك، فضع في اعتبارك أنه مع معظم الـ futures، لا ينبغي للمستدعي استدعاء `poll` مرة أخرى بعد أن تعيد الـ future القيمة `Ready`. العديد من الـ futures ستصاب بالذعر (panic) إذا تم استدعاء `poll` عليها مرة أخرى بعد أن تصبح جاهزة. الـ futures التي يكون من الآمن استدعاء `poll` عليها مرة أخرى ستذكر ذلك صراحة في توثيقها. هذا مشابه لكيفية سلوك `Iterator::next`.

عندما ترى كوداً يستخدم `await` ، تقوم Rust بتجميعه (compile) خلف الكواليس إلى كود يستدعي `poll`. إذا نظرت إلى القائمة 17-4، حيث قمنا بطباعة عنوان الصفحة لعنوان URL واحد بمجرد اكتماله، فإن Rust تقوم بتجميعه إلى شيء يشبه (وإن لم يكن مطابقاً تماماً) هذا:

```rust,ignore
match page_title(url).poll() {
    Ready(page_title) => match page_title {
        Some(title) => println!("The title for {url} was {title}"),
        None => println!("{url} had no title"),
    },
    Pending => {
        // ولكن ماذا نضع هنا؟
    }
}
```

ماذا يجب أن نفعل عندما لا تزال الـ future في حالة `Pending`؟ نحتاج إلى طريقة ما للمحاولة مراراً وتكراراً حتى تصبح الـ future جاهزة أخيراً. بعبارة أخرى، نحتاج إلى حلقة تكرار (loop):

```rust,ignore
let mut page_title_fut = page_title(url);
loop {
    match page_title_fut.poll() {
        Ready(value) => match page_title {
            Some(title) => println!("The title for {url} was {title}"),
            None => println!("{url} had no title"),
        },
        Pending => {
            // استمرار
        }
    }
}
```

ومع ذلك، إذا قامت Rust بتجميعه إلى ذلك الكود بالضبط، فإن كل `await` ستكون حاجزة (blocking) - وهو عكس ما كنا نسعى إليه تماماً! بدلاً من ذلك، تضمن Rust أن الـ loop يمكنها تسليم التحكم إلى شيء يمكنه إيقاف العمل مؤقتاً على هذه الـ future للعمل على futures أخرى ثم التحقق من هذه الـ future مرة أخرى لاحقاً. كما رأينا، هذا الشيء هو وقت تشغيل غير متزامن (async runtime)، وعمل الجدولة (scheduling) والتنسيق هذا هو أحد وظائفه الرئيسية.

في قسم ["إرسال البيانات بين مهمتين باستخدام تمرير الرسائل"][message-passing]، وصفنا الانتظار على `rx.recv`. استدعاء `recv` يعيد future، وانتظار الـ future يستدعي `poll` عليها. لاحظنا أن الـ runtime سيوقف الـ future مؤقتاً حتى تصبح جاهزة إما بـ `Some(message)` أو `None` عند إغلاق القناة (channel). مع فهمنا الأعمق لسمة `Future` ، وتحديداً `Future::poll` ، يمكننا رؤية كيفية عمل ذلك. يعرف الـ runtime أن الـ future ليست جاهزة عندما تعيد `Poll::Pending`. وعلى العكس من ذلك، يعرف الـ runtime أن الـ future _جاهزة_ ويقوم بتقديمها عندما تعيد `poll` القيمة `Poll::Ready(Some(message))` أو `Poll::Ready(None)`.

التفاصيل الدقيقة لكيفية قيام الـ runtime بذلك تقع خارج نطاق هذا الكتاب، ولكن المفتاح هو رؤية الآليات الأساسية للـ futures: يقوم الـ runtime باستدعاء _poll_ لكل future هو مسؤول عنها، ويعيد الـ future إلى وضع السكون عندما لا تكون جاهزة بعد.

<!-- Old headings. Do not remove or links may break. -->

<a id="pinning-and-the-pin-and-unpin-traits"></a>
<a id="the-pin-and-unpin-traits"></a>

### نوع `Pin` وسمة `Unpin`

بالعودة إلى القائمة 17-13، استخدمنا ماكرو (macro) `trpl::join!` لانتظار ثلاث futures. ومع ذلك، فمن الشائع أن يكون لديك مجموعة (collection) مثل متجه (vector) يحتوي على عدد من الـ futures التي لن تُعرف حتى وقت التشغيل (runtime). لنقم بتغيير القائمة 17-13 إلى الكود الموجود في القائمة 17-23 الذي يضع الـ futures الثلاثة في vector ويستدعي دالة `trpl::join_all` بدلاً من ذلك، وهو ما لن يتم تجميعه بعد.

<Listing number="17-23" caption="انتظار الـ futures في مجموعة"  file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-23/src/main.rs:here}}
```

</Listing>

لقد وضعنا كل future داخل `Box` لتحويلها إلى كائنات سمات (trait objects)، تماماً كما فعلنا في قسم "إرجاع الأخطاء من `run`" في الفصل 12. (سنغطي trait objects بالتفصيل في الفصل 18). يتيح لنا استخدام trait objects معاملة كل من الـ futures المجهولة الناتجة عن هذه الأنواع كأنها من نفس النوع، لأن جميعها تطبق سمة `Future`.

قد يكون هذا مفاجئاً. ففي النهاية، لا تعيد أي من الكتل غير المتزامنة (async blocks) أي شيء، لذا ينتج عن كل منها `Future<Output = ()>`. تذكر أن `Future` هي trait، وأن المترجم (compiler) ينشئ تعداداً (enum) فريداً لكل async block، حتى عندما يكون لها أنواع مخرجات متطابقة. تماماً كما لا يمكنك وضع هيكلين (structs) مختلفين مكتوبين يدوياً في `Vec` ، لا يمكنك خلط الـ enums التي أنشأها الـ compiler.

ثم نقوم بتمرير الـ collection الخاصة بالـ futures إلى دالة `trpl::join_all` وننتظر النتيجة. ومع ذلك، لا يتم تجميع هذا؛ إليك الجزء ذو الصلة من رسائل الخطأ.

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-23
cargo build
copy *only* the final `error` block from the errors
-->

```text
error[E0277]: `dyn Future<Output = ()>` cannot be unpinned
  --> src/main.rs:48:33
   |
48 |         trpl::join_all(futures).await;
   |                                 ^^^^^ the trait `Unpin` is not implemented for `dyn Future<Output = ()>`
   |
   = note: consider using the `pin!` macro
           consider using `Box::pin` if you need to access the pinned value outside of the current scope
   = note: required for `Box<dyn Future<Output = ()>>` to implement `Future`
note: required by a bound in `futures_util::future::join_all::JoinAll`
  --> file:///home/.cargo/registry/src/index.crates.io-1949cf8c6b5b557f/futures-util-0.3.30/src/future/join_all.rs:29:8
   |
27 | pub struct JoinAll<F>
   |            ------- required by a bound in this struct
28 | where
29 |     F: Future,
   |        ^^^^^^ required by this bound in `JoinAll`
```

تخبرنا الملاحظة في رسالة الخطأ هذه أنه يجب علينا استخدام macro `pin!` لـ "تثبيت" (pin) القيم، مما يعني وضعها داخل نوع `Pin` الذي يضمن عدم نقل القيم في الذاكرة (memory). تقول رسالة الخطأ أن التثبيت (pinning) مطلوب لأن `dyn Future<Output = ()>` تحتاج إلى تطبيق سمة `Unpin` وهي لا تفعل ذلك حالياً.

تعيد دالة `trpl::join_all` هيكلاً يسمى `JoinAll`. هذا الـ struct عام (generic) على نوع `F` ، وهو مقيد بتطبيق سمة `Future`. انتظار future مباشرة باستخدام `await` يثبت الـ future ضمنياً. لهذا السبب لا نحتاج إلى استخدام `pin!` في كل مكان نريد فيه انتظار futures.

ومع ذلك، نحن لا ننتظر future مباشرة هنا. بدلاً من ذلك، نقوم بإنشاء future جديدة، `JoinAll` ، عن طريق تمرير collection من الـ futures إلى دالة `join_all`. يتطلب توقيع (signature) دالة `join_all` أن تطبق جميع أنواع العناصر في الـ collection سمة `Future` ، ويطبق `Box<T>` سمة `Future` فقط إذا كان الـ `T` الذي يغلفه هو future تطبق سمة `Unpin`.

هذا الكثير لاستيعابه! لفهمه حقاً، دعونا نتعمق قليلاً في كيفية عمل سمة `Future` فعلياً، لا سيما فيما يتعلق بـ pinning. انظر مرة أخرى إلى تعريف سمة `Future`:

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

pub trait Future {
    type Output;

    // Required method
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

المعامل `cx` ونوعه `Context` هما المفتاح لكيفية معرفة الـ runtime فعلياً متى يجب التحقق من أي future معينة مع بقائها كسولة (lazy). مرة أخرى، تفاصيل كيفية عمل ذلك تقع خارج نطاق هذا الفصل، وعادة ما تحتاج فقط إلى التفكير في هذا عند كتابة تطبيق `Future` مخصص. سنركز بدلاً من ذلك على نوع `self` ، حيث أن هذه هي المرة الأولى التي نرى فيها method حيث يكون لـ `self` تعليق توضيحي للنوع (type annotation). يعمل الـ type annotation لـ `self` مثل الـ type annotations لمعاملات الدوال الأخرى ولكن مع اختلافين رئيسيين:

- يخبر Rust بنوع `self` الذي يجب أن يكون عليه لاستدعاء الـ method.
- لا يمكن أن يكون أي نوع فحسب. فهو مقتصر على النوع الذي تم تطبيق الـ method عليه، أو مرجع أو مؤشر ذكي (smart pointer) لهذا النوع، أو `Pin` يغلف مرجعاً لهذا النوع.

سنرى المزيد عن هذا الـ syntax في [الفصل 18][ch-18]. في الوقت الحالي، يكفي أن نعرف أنه إذا أردنا استدعاء poll على future للتحقق مما إذا كانت `Pending` أو `Ready(Output)` ، فنحن بحاجة إلى مرجع قابل للتغيير مغلف بـ `Pin` للنوع.

`Pin` هو غلاف لأنواع تشبه المؤشرات (pointer-like types) مثل `&` و `&mut` و `Box` و `Rc`. (من الناحية الفنية، يعمل `Pin` مع الأنواع التي تطبق سمات `Deref` أو `DerefMut` ، ولكن هذا يعادل فعلياً العمل فقط مع الـ references والـ smart pointers). الـ `Pin` ليس مؤشراً بحد ذاته وليس له أي سلوك خاص به مثل `Rc` و `Arc` مع عد المراجع (reference counting)؛ إنه مجرد أداة يستخدمها الـ
```
لماذا يحتاج `self` إلى أن يكون في نوع `Pin` لاستدعاء `poll`؟

تذكر مما سبق في هذا الفصل أن سلسلة من نقاط الانتظار (await points) في الـ future يتم تجميعها في آلة حالة (state machine)، ويتأكد المترجم من أن هذه الـ state machine تتبع جميع قواعد Rust المعتادة حول السلامة، بما في ذلك الاستعارة (borrowing) والملكية (ownership). لإنجاح ذلك، تنظر Rust إلى البيانات المطلوبة بين await point واحدة والـ await point التالية أو نهاية الكتلة غير المتزامنة (async block). ثم تنشئ متغيراً (variant) مقابلاً في الـ state machine المجمعة. يحصل كل variant على الوصول الذي يحتاجه للبيانات التي سيتم استخدامها في ذلك القسم من الكود المصدري، سواء عن طريق أخذ ملكية تلك البيانات أو الحصول على مرجع قابل للتغيير (mutable reference) أو غير قابل للتغيير (immutable reference) لها.

حتى الآن، كل شيء يسير على ما يرام: إذا أخطأنا في أي شيء يتعلق بالملكية أو المراجع في async block معينة، فسيخبرنا مدقق الاستعارة (borrow checker). ولكن عندما نريد نقل الـ future المقابلة لتلك الكتلة - مثل نقلها إلى `Vec` لتمريرها إلى `join_all` - تصبح الأمور أكثر تعقيداً.

عندما ننقل future - سواء عن طريق دفعها إلى هيكل بيانات لاستخدامها كمكرر (iterator) مع `join_all` أو عن طريق إعادتها من دالة - فإن ذلك يعني فعلياً نقل الـ state machine التي تنشئها Rust لنا. وعلى عكس معظم الأنواع الأخرى في Rust، يمكن للـ futures التي تنشئها Rust لـ async blocks أن تنتهي بمراجع لنفسها في حقول أي variant معين، كما هو موضح في الرسم التوضيحي المبسط في الشكل 17-4.

<figure>

<img alt="A single-column, three-row table representing a future, fut1, which has data values 0 and 1 in the first two rows and an arrow pointing from the third row back to the second row, representing an internal reference within the future." src="img/trpl17-04.svg" class="center" />

<figcaption>الشكل 17-4: نوع بيانات مرجعي ذاتي (self-referential data type)</figcaption>

</figure>

بشكل افتراضي، يكون أي كائن له مرجع لنفسه غير آمن للنقل، لأن المراجع تشير دائماً إلى عنوان الذاكرة (memory address) الفعلي لأي شيء تشير إليه (انظر الشكل 17-5). إذا قمت بنقل هيكل البيانات نفسه، فستبقى تلك المراجع الداخلية تشير إلى الموقع القديم. ومع ذلك، أصبح موقع الذاكرة هذا الآن غير صالح. لسبب واحد، لن يتم تحديث قيمته عندما تجري تغييرات على هيكل البيانات. ولسبب آخر - وهو الأكثر أهمية - أصبح الكمبيوتر الآن حراً في إعادة استخدام تلك الذاكرة لأغراض أخرى! قد ينتهي بك الأمر بقراءة بيانات غير ذات صلة تماماً لاحقاً.

<figure>

<img alt="Two tables, depicting two futures, fut1 and fut2, each of which has one column and three rows, representing the result of having moved a future out of fut1 into fut2. The first, fut1, is grayed out, with a question mark in each index, representing unknown memory. The second, fut2, has 0 and 1 in the first and second rows and an arrow pointing from its third row back to the second row of fut1, representing a pointer that is referencing the old location in memory of the future before it was moved." src="img/trpl17-05.svg" class="center" />

<figcaption>الشكل 17-5: النتيجة غير الآمنة لنقل نوع بيانات مرجعي ذاتي</figcaption>

</figure>

نظرياً، يمكن لمترجم Rust محاولة تحديث كل مرجع لكائن كلما تم نقله، ولكن هذا قد يضيف الكثير من العبء على الأداء، خاصة إذا كانت هناك شبكة كاملة من المراجع تحتاج إلى تحديث. إذا تمكنا بدلاً من ذلك من التأكد من أن هيكل البيانات المعني _لا يتحرك في الذاكرة_، فلن نضطر إلى تحديث أي مراجع. هذا هو بالضبط الغرض من borrow checker في Rust: في الكود الآمن، يمنعك من نقل أي عنصر له مرجع نشط يشير إليه.

يعتمد `Pin` على ذلك ليعطينا الضمان الدقيق الذي نحتاجه. عندما نقوم بـ "تثبيت" (pin) قيمة عن طريق تغليف مؤشر لتلك القيمة في `Pin` ، فإنها لا تعود قادرة على الحركة. وبالتالي، إذا كان لديك `Pin<Box<SomeType>>` ، فأنت في الواقع تثبت قيمة `SomeType` ، _وليس_ مؤشر `Box`. يوضح الشكل 17-6 هذه العملية.

<figure>

<img alt="Three boxes laid out side by side. The first is labeled “Pin”, the second “b1”, and the third “pinned”. Within “pinned” is a table labeled “fut”, with a single column; it represents a future with cells for each part of the data structure. Its first cell has the value “0”, its second cell has an arrow coming out of it and pointing to the fourth and final cell, which has the value “1” in it, and the third cell has dashed lines and an ellipsis to indicate there may be other parts to the data structure. All together, the “fut” table represents a future which is self-referential. An arrow leaves the box labeled “Pin”, goes through the box labeled “b1” and terminates inside the “pinned” box at the “fut” table." src="img/trpl17-06.svg" class="center" />

<figcaption>الشكل 17-6: تثبيت `Box` يشير إلى نوع future مرجعي ذاتي</figcaption>

</figure>

في الواقع، لا يزال بإمكان مؤشر `Box` التحرك بحرية. تذكر: نحن نهتم بالتأكد من أن البيانات التي يتم الرجوع إليها في النهاية تبقى في مكانها. إذا تحرك المؤشر، _ولكن البيانات التي يشير إليها_ بقيت في نفس المكان، كما في الشكل 17-7، فلا توجد مشكلة محتملة. (كتمرين مستقل، انظر إلى توثيقات الأنواع بالإضافة إلى وحدة `std::pin` وحاول معرفة كيف يمكنك القيام بذلك باستخدام `Pin` يغلف `Box`). المفتاح هو أن النوع المرجعي الذاتي نفسه لا يمكنه التحرك، لأنه لا يزال مثبتاً.

<figure>

<img alt="Four boxes laid out in three rough columns, identical to the previous diagram with a change to the second column. Now there are two boxes in the second column, labeled “b1” and “b2”, “b1” is grayed out, and the arrow from “Pin” goes through “b2” instead of “b1”, indicating that the pointer has moved from “b1” to “b2”, but the data in “pinned” has not moved." src="img/trpl17-07.svg" class="center" />

<figcaption>الشكل 17-7: نقل `Box` يشير إلى نوع future مرجعي ذاتي</figcaption>

</figure>

ومع ذلك، فإن معظم الأنواع آمنة تماماً للنقل، حتى لو كانت خلف مؤشر `Pin`. نحتاج فقط إلى التفكير في التثبيت عندما تحتوي العناصر على مراجع داخلية. القيم البدائية (primitive values) مثل الأرقام والقيم المنطقية (Booleans) آمنة لأنها بوضوح لا تحتوي على أي مراجع داخلية.
وكذلك معظم الأنواع التي تتعامل معها عادةً في Rust. يمكنك نقل `Vec` ، على سبيل المثال، دون قلق. بالنظر إلى ما رأيناه حتى الآن، إذا كان لديك `Pin<Vec<String>>` ، فسيتعين عليك القيام بكل شيء عبر واجهات برمجة التطبيقات (APIs) الآمنة ولكن المقيدة التي يوفرها `Pin` ، على الرغم من أن `Vec<String>` آمن دائماً للنقل إذا لم تكن هناك مراجع أخرى له. نحتاج إلى طريقة لإخبار المترجم أنه من الجيد نقل العناصر في حالات مثل هذه - وهنا يأتي دور سمة التمييز (marker trait) المسماة `Unpin`.

الـ `Unpin` هي marker trait، مشابهة لسمات `Send` و `Sync` التي رأيناها في الفصل 16، وبالتالي ليس لها وظائف خاصة بها. توجد سمات التمييز فقط لإخبار المترجم أنه من الآمن استخدام النوع الذي يطبق سمة معينة في سياق معين. تخبر `Unpin` المترجم أن نوعاً معيناً _لا_ يحتاج إلى الالتزام بأي ضمانات حول ما إذا كان يمكن نقل القيمة المعنية بأمان.

<!--
  The inline `<code>` in the next block is to allow the inline `<em>` inside it,
  matching what NoStarch does style-wise, and emphasizing within the text here
  that it is something distinct from a normal type.
-->

تماماً كما هو الحال مع `Send` و `Sync` ، يطبق المترجم `Unpin` تلقائياً لجميع الأنواع التي يمكنه إثبات أنها آمنة. وهناك حالة خاصة، مشابهة أيضاً لـ `Send` و `Sync` ، وهي عندما _لا_ يتم تطبيق `Unpin` لنوع ما. الصيغة لهذا هي <code>impl !Unpin for <em>SomeType</em></code>، حيث <code><em>SomeType</em></code> هو اسم النوع الذي _يحتاج_ إلى الالتزام بتلك الضمانات ليكون آمناً كلما تم استخدام مؤشر لهذا النوع في `Pin`.

بمعنى آخر، هناك شيئان يجب وضعهما في الاعتبار حول العلاقة بين `Pin` و `Unpin`. أولاً، `Unpin` هي الحالة "العادية"، و `!Unpin` هي الحالة الخاصة. ثانياً، ما إذا كان النوع يطبق `Unpin` أو `!Unpin` يهم _فقط_ عندما تستخدم مؤشراً مثبتاً لهذا النوع مثل <code>Pin<&mut <em>SomeType</em>></code>.

لجعل ذلك ملموساً، فكر في `String`: لها طول وحروف Unicode التي تشكلها. يمكننا تغليف `String` في `Pin` ، كما هو موضح في الشكل 17-8. ومع ذلك، تطبق `String` تلقائياً `Unpin` ، كما تفعل معظم الأنواع الأخرى في Rust.

<figure>

<img alt="A box labeled “Pin” on the left with an arrow going from it to a box labeled “String” on the right. The “String” box contains the data 5usize, representing the length of the string, and the letters “h”, “e”, “l”, “l”, and “o” representing the characters of the string “hello” stored in this String instance. A dotted rectangle surrounds the “String” box and its label, but not the “Pin” box." src="img/trpl17-08.svg" class="center" />

<figcaption>الشكل 17-8: تثبيت `String`؛ يشير الخط المنقط إلى أن الـ `String` تطبق سمة `Unpin` وبالتالي فهي ليست مثبتة</figcaption>

</figure>

ونتيجة لذلك، يمكننا القيام بأشياء قد تكون غير قانونية إذا كانت `String` تطبق `!Unpin` بدلاً من ذلك، مثل استبدال سلسلة نصية بأخرى في نفس موقع الذاكرة تماماً كما في الشكل 17-9. هذا لا ينتهك عقد `Pin` ، لأن `String` ليس لها مراجع داخلية تجعل من غير الآمن نقلها. وهذا هو بالضبط سبب تطبيقها لـ `Unpin` بدلاً من `!Unpin`.

<figure>

<img alt="The same “hello” string data from the previous example, now labeled “s1” and grayed out. The “Pin” box from the previous example now points to a different String instance, one that is labeled “s2”, is valid, has a length of 7usize, and contains the characters of the string “goodbye”. s2 is surrounded by a dotted rectangle because it, too, implements the Unpin trait." src="img/trpl17-09.svg" class="center" />

<figcaption>الشكل 17-9: استبدال الـ `String` بـ `String` مختلفة تماماً في الذاكرة</figcaption>

</figure>

الآن نعرف ما يكفي لفهم الأخطاء المبلغ عنها لاستدعاء `join_all` من القائمة 17-23. حاولنا في الأصل نقل الـ futures الناتجة عن async blocks إلى `Vec<Box<dyn Future<Output = ()>>>` ، ولكن كما رأينا، قد تحتوي تلك الـ futures على مراجع داخلية، لذا فهي لا تطبق `Unpin` تلقائياً. بمجرد تثبيتها، يمكننا تمرير نوع `Pin` الناتج إلى الـ `Vec` ، ونحن واثقون من أن البيانات الأساسية في الـ futures _لن_ يتم نقلها. توضح القائمة 17-24 كيفية إصلاح الكود عن طريق استدعاء macro `pin!` حيث يتم تعريف كل من الـ futures الثلاثة وتعديل نوع trait object.

<Listing number="17-24" caption="تثبيت الـ futures لتمكين نقلها إلى المتجه">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-24/src/main.rs:here}}
```

</Listing>

هذا المثال الآن يتم تجميعه وتشغيله، ويمكننا إضافة أو إزالة futures من الـ vector في وقت التشغيل وضمها جميعاً.

تعتبر `Pin` و `Unpin` مهمة في الغالب لبناء مكتبات منخفضة المستوى، أو عندما تبني runtime بنفسك، بدلاً من كود Rust اليومي. ولكن عندما ترى هذه السمات في رسائل الخطأ، سيكون لديك الآن فكرة أفضل عن كيفية إصلاح الكود الخاص بك!

> ملاحظة: هذا المزيج من `Pin` و `Unpin` يجعل من الممكن تطبيق فئة كاملة من الأنواع المعقدة في Rust بأمان والتي قد تكون صعبة لولا ذلك لأنها مرجعية ذاتية. تظهر الأنواع التي تتطلب `Pin` بشكل شائع في Rust غير المتزامن اليوم، ولكن بين الحين والآخر، قد تراها في سياقات أخرى أيضاً.
>
> تفاصيل كيفية عمل `Pin` و `Unpin` ، والقواعد التي يتعين عليهما الالتزام بها، مغطاة على نطاق واسع في توثيق API لـ `std::pin` ، لذا إذا كنت مهتماً بمعرفة المزيد، فهذا مكان رائع للبدء.
>
> إذا كنت تريد فهم كيفية عمل الأشياء تحت الغطاء بمزيد من التفصيل، فراجع الفصلين [2][under-the-hood] و [4][pinning] من كتاب [_البرمجة غير المتزامنة في Rust_][async-book].

### سمة `Stream`

الآن بعد أن أصبح لديك فهم أعمق لسمات `Future` و `Pin` و `Unpin` ، يمكننا توجيه انتباهنا إلى سمة `Stream`. كما تعلمت سابقاً في هذا الفصل، فإن الـ streams تشبه المكررات غير المتزامنة (asynchronous iterators). ومع ذلك، على عكس `Iterator` و `Future` ، لا يوجد تعريف لـ `Stream` في المكتبة القياسية (standard library) حتى وقت كتابة هذا التقرير، ولكن _يوجد_ تعريف شائع جداً من حزمة (crate) تسمى `futures` يُستخدم في جميع أنحاء النظام البيئي.

دعونا نراجع تعريفات سمات `Iterator` و `Future` قبل النظر في كيفية قيام سمة `Stream` بدمجهما معاً. من `Iterator` ، لدينا فكرة التسلسل: توفر طريقة `next` الخاصة بها `Option<Self::Item>`. ومن `Future` ، لدينا فكرة الجاهزية بمرور الوقت: توفر طريقة `poll` الخاصة بها `Poll<Self::Output>`. لتمثيل تسلسل من العناصر التي تصبح جاهزة بمرور الوقت، نحدد سمة `Stream` التي تجمع تلك الميزات معاً:

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

trait Stream {
    type Item;
```
<!--
TODO: update this if/when tokio/etc. update their MSRV and switch to using async functions
in traits, since the lack thereof is the reason they do not yet have this.
-->

> ملاحظة: التعريف الفعلي الذي استخدمناه سابقاً في هذا الفصل يبدو مختلفاً قليلاً عن هذا، لأنه يدعم إصدارات Rust التي لم تكن تدعم بعد استخدام الدوال غير المتزامنة (async functions) في السمات (traits). ونتيجة لذلك، يبدو كالتالي:
>
> ```rust,ignore
> fn next(&mut self) -> Next<'_, Self> where Self: Unpin;
> ```
>
> نوع `Next` هذا هو هيكل (struct) يطبق `Future` ويسمح لنا بتسمية عمر (lifetime) المرجع لـ `self` بـ `Next<'_, Self>` ، بحيث يمكن لـ `await` العمل مع هذه الطريقة (method).

سمة `StreamExt` هي أيضاً موطن لجميع الـ methods المثيرة للاهتمام المتاحة للاستخدام مع الـ streams. يتم تطبيق `StreamExt` تلقائياً لكل نوع يطبق `Stream` ، ولكن يتم تعريف هذه الـ traits بشكل منفصل لتمكين المجتمع من تطوير واجهات برمجة تطبيقات (APIs) مريحة دون التأثير على الـ trait الأساسية.

في إصدار `StreamExt` المستخدم في حزمة `trpl` ، لا يحدد الـ trait طريقة `next` فحسب، بل يوفر أيضاً تطبيقاً افتراضياً لـ `next` يتعامل بشكل صحيح مع تفاصيل استدعاء `Stream::poll_next`. هذا يعني أنه حتى عندما تحتاج إلى كتابة نوع بيانات تدفقي (streaming data type) خاص بك، فإنك _فقط_ تضطر إلى تطبيق `Stream` ، ومن ثم يمكن لأي شخص يستخدم نوع البيانات الخاص بك استخدام `StreamExt` وطرقها معه تلقائياً.

هذا كل ما سنغطيه فيما يتعلق بالتفاصيل منخفضة المستوى حول هذه الـ traits. وللختام، دعونا نفكر في كيفية ترابط الـ futures (بما في ذلك الـ streams) والمهام (tasks) والخيوط (threads) معاً!

[message-passing]: ch17-02-concurrency-with-async.md#sending-data-between-two-tasks-using-message-passing
[ch-18]: ch18-00-oop.html
[async-book]: https://rust-lang.github.io/async-book/
[under-the-hood]: https://rust-lang.github.io/async-book/02_execution/01_chapter.html
[pinning]: https://rust-lang.github.io/async-book/04_pinning/01_chapter.html
[first-async]: ch17-01-futures-and-syntax.html#our-first-async-program
[any-number-futures]: ch17-03-more-futures.html#working-with-any-number-of-futures
[streams]: ch17-04-streams.html
```
