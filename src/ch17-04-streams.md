<!-- Old headings. Do not remove or links may break. -->

<a id="streams"></a>

## التدفقات: المستقبلات في تسلسل (Streams: Futures in Sequence)

تذكر كيف استخدمنا المستلم (receiver) لقناتنا غير المتزامنة (async channel) في وقت سابق من هذا الفصل في قسم ["تمرير الرسائل"][17-02-messages]. تنتج دالة `recv` غير المتزامنة تسلسلاً (sequence) من العناصر بمرور الوقت. هذا مثال على نمط (pattern) أكثر عمومية يعرف باسم التدفق (stream). يتم تمثيل العديد من المفاهيم بشكل طبيعي كـ تدفقات (streams): العناصر التي تصبح متاحة في طابور (queue)، أو قطع (chunks) البيانات التي يتم سحبها تدريجياً من نظام الملفات (filesystem) عندما تكون مجموعة البيانات الكاملة كبيرة جداً بالنسبة لـ ذاكرة (memory) الكمبيوتر، أو البيانات التي تصل عبر الشبكة (network) بمرور الوقت. ولأن الـ streams هي مستقبلات (futures)، يمكننا استخدامها مع أي نوع آخر من الـ future ودمجها بطرق مثيرة للاهتمام. على سبيل المثال، يمكننا تجميع (batch up) الأحداث (events) لتجنب إطلاق الكثير من استدعاءات الشبكة (network calls)، أو تعيين مهلات زمنية (timeouts) على تسلسلات من العمليات (operations) طويلة الأمد، أو تقييد (throttle) أحداث واجهة المستخدم (user interface) لتجنب القيام بعمل غير ضروري.

لقد رأينا sequence من العناصر في الفصل الثالث عشر، عندما نظرنا إلى سمة (trait) المكرر (iterator) في قسم ["سمة Iterator ودالة `next`"][iterator-trait]، ولكن هناك فرقان بين الـ iterators ومستلم الـ async channel. الفرق الأول هو الوقت: الـ iterators متزامنة (synchronous)، بينما مستلم الـ channel غير متزامن (asynchronous). الفرق الثاني هو واجهة برمجة التطبيقات (API). عند العمل مباشرة مع `Iterator` ، فإننا نستدعي دالة `next` الـ synchronous الخاصة به. أما مع stream الـ `trpl::Receiver` بشكل خاص، فقد استدعينا دالة `recv` الـ asynchronous بدلاً من ذلك. بخلاف ذلك، تبدو هذه الـ APIs متشابهة جداً، وهذا التشابه ليس صدفة. الـ stream يشبه شكلاً asynchronous من التكرار (iteration). وبينما ينتظر `trpl::Receiver` تحديداً لاستلام الرسائل، فإن الـ API العام للـ stream أوسع بكثير: فهو يوفر العنصر التالي بالطريقة التي يفعلها `Iterator` ، ولكن بشكل asynchronous.

يعني التشابه بين الـ iterators والـ streams في Rust أنه يمكننا بالفعل إنشاء stream من أي iterator. كما هو الحال مع iterator، يمكننا العمل مع stream من خلال استدعاء دالة `next` الخاصة به ثم انتظار (awaiting) المخرجات، كما في القائمة 17-21، والتي لن يتم تجميعها (compile) بعد.

<Listing number="17-21" caption="Creating a stream from an iterator and printing its values" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-21/src/main.rs:stream}}
```

</Listing>

نبدأ بـ مصفوفة (array) من الأرقام، والتي نحولها إلى مكرر (iterator) ثم نستدعي `map` عليه لمضاعفة جميع القيم. ثم نحول الـ iterator إلى stream باستخدام دالة `trpl::stream_from_iter`. بعد ذلك، نقوم بعمل حلقة (loop) على العناصر في الـ stream أثناء وصولها باستخدام حلقة `while let`.

للأسف، عندما نحاول تشغيل الكود، فإنه لا يتم تجميعه بل يبلغ بدلاً من ذلك عن عدم وجود دالة `next` متاحة:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-21
cargo build
copy only the error output
-->

```text
error[E0599]: no method named `next` found for struct `tokio_stream::iter::Iter` in the current scope
  --> src/main.rs:10:40
   |
10 |         while let Some(value) = stream.next().await {
   |                                        ^^^^
   |
   = help: items from traits can only be used if the trait is in scope
help: the following traits which provide `next` are implemented but not in scope; perhaps you want to import one of them
   |
1  + use crate::trpl::StreamExt;
   |
1  + use futures_util::stream::stream::StreamExt;
   |
1  + use std::iter::Iterator;
   |
1  + use std::str::pattern::Searcher;
   |
help: there is a method `try_next` with a similar name
   |
10 |         while let Some(value) = stream.try_next().await {
   |                                        ~~~~~~~~
```

كما يوضح هذا المخرج، فإن سبب خطأ المترجم (compiler) هو أننا بحاجة إلى الـ trait الصحيح في النطاق (scope) لنتمكن من استخدام دالة `next`. بالنظر إلى نقاشنا حتى الآن، قد تتوقع بشكل معقول أن يكون هذا الـ trait هو `Stream` ، ولكنه في الواقع `StreamExt`. وهو اختصار لـ امتداد (extension)، و `Ext` هو نمط شائع في مجتمع Rust لتوسيع trait بآخر.

تحدد سمة `Stream` واجهة (interface) منخفضة المستوى تجمع بشكل فعال بين سمات `Iterator` و `Future`. توفر `StreamExt` مجموعة من الـ APIs عالية المستوى فوق `Stream` ، بما في ذلك دالة `next` بالإضافة إلى دوال مساعدة (utility methods) أخرى مشابهة لتلك التي توفرها سمة `Iterator`. لم تصبح `Stream` و `StreamExt` بعد جزءاً من المكتبة القياسية (standard library) لـ Rust، ولكن معظم حزم النظام البيئي (ecosystem crates) تستخدم تعريفات مماثلة.

إصلاح خطأ الـ compiler هو إضافة عبارة `use` لـ `trpl::StreamExt` ، كما في القائمة 17-22.

<Listing number="17-22" caption="Successfully using an iterator as the basis for a stream" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-22/src/main.rs:all}}
```

</Listing>

مع وضع كل تلك القطع معاً، يعمل هذا الكود بالطريقة التي نريدها! علاوة على ذلك، الآن بعد أن أصبح لدينا `StreamExt` في الـ scope، يمكننا استخدام جميع الـ utility methods الخاصة به، تماماً كما هو الحال مع الـ iterators.

[17-02-messages]: ch17-02-concurrency-with-async.html#message-passing
[iterator-trait]: ch13-02-iterators.html#the-iterator-trait-and-the-next-method
