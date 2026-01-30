## الملحق ب: (المعاملات) Operators و (الرموز) Symbols

يحتوي هذا الملحق على مسرد لـ (الصيغة) `Syntax` الخاصة بلغة Rust، بما في ذلك (المعاملات) `Operators` و (الرموز) `Symbols` الأخرى التي تظهر بمفردها أو في سياق (المسارات) `Paths`، و (الأنواع العامة) `Generics`، و (قيود السمات) `Trait bounds`، و (الماكروهات) `Macros`، و (السمات) `Attributes`، و (التعليقات) `Comments`، و (المجموعات) `Tuples`، و (الأقواس) `Brackets`.

### Operators

يحتوي الجدول B-1 على `Operators` في Rust، ومثال على كيفية ظهور `Operator` في السياق، وشرح موجز، وما إذا كان `Operator` (قابل للتحميل الزائد) `Overloadable`. إذا كان `Operator` `Overloadable`، يتم إدراج `Trait` ذي الصلة الذي يجب استخدامه لتحميل هذا `Operator` الزائد.

<span class="caption">الجدول B-1: Operators</span>

| Operator | مثال | الشرح | Overloadable؟ |
| :--- | :--- | :--- | :--- |
| `!` | `ident!(...)`, `ident!{...}`, `ident![...]` | (توسيع الماكرو) `Macro expansion` | |
| `!` | `!expr` | (المكمل المنطقي أو على مستوى البت) `Bitwise or logical complement` | `Not` |
| `!=` | `expr != expr` | (مقارنة عدم التساوي) `Nonequality comparison` | `PartialEq` |
| `%` | `expr % expr` | (باقي القسمة الحسابي) `Arithmetic remainder` | `Rem` |
| `%=` | `var %= expr` | (باقي القسمة الحسابي والإسناد) `Arithmetic remainder and assignment` | `RemAssign` |
| `&` | `&expr`, `&mut expr` | (الاستعارة) `Borrow` | |
| `&` | `&type`, `&mut type`, `&'a type`, `&'a mut type` | (نوع المؤشر المستعار) `Borrowed pointer type` | |
| `&` | `expr & expr` | (البتي AND) `Bitwise AND` | `BitAnd` |
| `&=` | `var &= expr` | (البتي AND والإسناد) `Bitwise AND and assignment` | `BitAndAssign` |
| `&&` | `expr && expr` | (المنطقي AND ذو الدائرة القصيرة) `Short-circuiting logical AND` | |
| `*` | `expr * expr` | (الضرب الحسابي) `Arithmetic multiplication` | `Mul` |
| `*=` | `var *= expr` | (الضرب الحسابي والإسناد) `Arithmetic multiplication and assignment` | `MulAssign` |
| `*` | `*expr` | (إلغاء الإشارة) `Dereference` | `Deref` |
| `*` | `*const type`, `*mut type` | (المؤشر الخام) `Raw pointer` | |
| `+` | `trait + trait`, `'a + trait` | (قيد النوع المركب) `Compound type constraint` | |
| `+` | `expr + expr` | (الجمع الحسابي) `Arithmetic addition` | `Add` |
| `+=` | `var += expr` | (الجمع الحسابي والإسناد) `Arithmetic addition and assignment` | `AddAssign` |
| `,` | `expr, expr` | (فاصل الوسائط والعناصر) `Argument and element separator` | |
| `-` | `- expr` | (النفي الحسابي) `Arithmetic negation` | `Neg` |
| `-` | `expr - expr` | (الطرح الحسابي) `Arithmetic subtraction` | `Sub` |
| `-=` | `var -= expr` | (الطرح الحسابي والإسناد) `Arithmetic subtraction and assignment` | `SubAssign` |
| `->` | `fn(...) -> type`, <code>&vert;...&vert; -> type</code> | (نوع الإرجاع للدالة والإغلاق) `Function and closure return type` | |
| `.` | `expr.ident` | (الوصول إلى الحقل) `Field access` | |
| `.` | `expr.ident(expr, ...)` | (استدعاء الدالة) `Method call` | |
| `.` | `expr.0`, `expr.1`, and so on | (فهرسة المجموعة) `Tuple indexing` | |
| `..` | `..`, `expr..`, `..expr`, `expr..expr` | (حرفي النطاق غير الشامل للطرف الأيمن) `Right-exclusive range literal` | `PartialOrd` |
| `..=` | `..=expr`, `expr..=expr` | (حرفي النطاق الشامل للطرف الأيمن) `Right-inclusive range literal` | `PartialOrd` |
| `..` | `..expr` | (صيغة تحديث حرفي الهيكل) `Struct literal update syntax` | |
| `..` | `variant(x, ..)`, `struct_type { x, .. }` | (ربط النمط "والباقي") `"And the rest" pattern binding` | |
| `...` | `expr...expr` | (مهمل، استخدم `..=` بدلاً منه) في نمط: (نمط النطاق الشامل) `inclusive range pattern` | |
| `/` | `expr / expr` | (القسمة الحسابية) `Arithmetic division` | `Div` |
| `/=` | `var /= expr` | (القسمة الحسابية والإسناد) `Arithmetic division and assignment` | `DivAssign` |
| `:` | `pat: type`, `ident: type` | (القيود) `Constraints` | |
| `:` | `ident: expr` | (مهيئ حقل الهيكل) `Struct field initializer` | |
| `:` | `'a: loop {...}` | (تسمية الحلقة) `Loop label` | |
| `;` | `expr;` | (مُنهي العبارة والعنصر) `Statement and item terminator` | |
| `;` | `[...; len]` | جزء من (صيغة المصفوفة ذات الحجم الثابت) `fixed-size array syntax` | |
| `<<` | `expr << expr` | (الإزاحة لليسار) `Left-shift` | `Shl` |
| `<<=` | `var <<= expr` | (الإزاحة لليسار والإسناد) `Left-shift and assignment` | `ShlAssign` |
| `<` | `expr < expr` | (مقارنة أقل من) `Less than comparison` | `PartialOrd` |
| `<=` | `expr <= expr` | (مقارنة أقل من أو يساوي) `Less than or equal to comparison` | `PartialOrd` |
| `=` | `var = expr`, `ident = type` | (الإسناد/التكافؤ) `Assignment/equivalence` | |
| `==` | `expr == expr` | (مقارنة التساوي) `Equality comparison` | `PartialEq` |
| `=>` | `pat => expr` | جزء من (صيغة ذراع المطابقة) `match arm syntax` | |
| `>` | `expr > expr` | (مقارنة أكبر من) `Greater than comparison` | `PartialOrd` |
| `>=` | `expr >= expr` | (مقارنة أكبر من أو يساوي) `Greater than or equal to comparison` | `PartialOrd` |
| `>>` | `expr >> expr` | (الإزاحة لليمين) `Right-shift` | `Shr` |
| `>>=` | `var >>= expr` | (الإزاحة لليمين والإسناد) `Right-shift and assignment` | `ShrAssign` |
| `@` | `ident @ pat` | (ربط النمط) `Pattern binding` | |
| `^` | `expr ^ expr` | (البتي OR الحصري) `Bitwise exclusive OR` | `BitXor` |
| `^=` | `var ^= expr` | (البتي OR الحصري والإسناد) `Bitwise exclusive OR and assignment` | `BitXorAssign` |
| <code>&vert;</code> | <code>pat &vert; pat</code> | (بدائل النمط) `Pattern alternatives` | |
| <code>&vert;</code> | <code>expr &vert; expr</code> | (البتي OR) `Bitwise OR` | `BitOr` |
| <code>&vert;=</code> | <code>var &vert;= expr</code> | (البتي OR والإسناد) `Bitwise OR and assignment` | `BitOrAssign` |
| <code>&vert;&vert;</code> | <code>expr &vert;&vert; expr</code> | (المنطقي OR ذو الدائرة القصيرة) `Short-circuiting logical OR` | |
| `?` | `expr?` | (نشر الخطأ) `Error propagation` | |

### Non-operator Symbols

تحتوي الجداول التالية على جميع `Symbols` التي لا تعمل كـ `Operators`؛ أي أنها لا تتصرف مثل استدعاء دالة أو `Method call`.

يعرض الجدول B-2 `Symbols` التي تظهر بمفردها وتكون صالحة في مجموعة متنوعة من المواقع.

<span class="caption">الجدول B-2: Stand-alone Syntax</span>

| Symbol | الشرح |
| :--- | :--- |
| `'ident` | (عمر مسمى أو تسمية حلقة) `Named lifetime or loop label` |
| Digits immediately followed by `u8`, `i32`, `f64`, `usize`, and so on | (حرفي رقمي من نوع محدد) `Numeric literal of specific type` |
| `"..."` | (حرفي سلسلة نصية) `String literal` |
| `r"..."`, `r#"..."#`, `r##"..."##`, and so on | (حرفي سلسلة نصية خام) `Raw string literal`؛ لا تتم معالجة (أحرف الهروب) `escape characters` |
| `b"..."` | (حرفي سلسلة بايت) `Byte string literal`؛ ينشئ (مصفوفة من البايتات) `array of bytes` بدلاً من `String` |
| `br"..."`, `br#"..."#`, `br##"..."##`, and so on | (حرفي سلسلة بايت خام) `Raw byte string literal`؛ مزيج من `Raw` و `Byte string literal` |
| `'...'` | (حرفي محرف) `Character literal` |
| `b'...'` | (حرفي بايت ASCII) `ASCII byte literal` |
| <code>&vert;...&vert; expr</code> | (الإغلاق) `Closure` |
| `!` | (النوع السفلي الفارغ دائمًا للدوال المتباينة) `Always-empty bottom type for diverging functions` |
| `_` | (ربط النمط "المتجاهل") `"Ignored" pattern binding`؛ يستخدم أيضًا لجعل (الحرفيات الصحيحة) `integer literals` قابلة للقراءة |

يعرض الجدول B-3 `Symbols` التي تظهر في سياق `Path` عبر (التسلسل الهرمي للوحدة) `module hierarchy` إلى (عنصر) `item`.

<span class="caption">الجدول B-3: Path-Related Syntax</span>

| Symbol | الشرح |
| :--- | :--- |
| `ident::ident` | (مسار مساحة الاسم) `Namespace path` |
| `super::path` | `Path` بالنسبة إلى (الوالد) `parent` للوحدة الحالية |
| `type::ident`, `<type as trait>::ident` | (الثوابت والدوال والأنواع المرتبطة) `Associated constants, functions, and types` |
| `<type>::...` | (العنصر المرتبط) `Associated item` لـ `type` لا يمكن تسميته مباشرة (على سبيل المثال، `<&T>::...`، `<[T]>::...`، وهكذا) |
| `trait::method(...)` | (إزالة الغموض) `Disambiguating` عن `Method call` عن طريق تسمية `Trait` الذي يحدده |
| `type::method(...)` | `Disambiguating` عن `Method call` عن طريق تسمية `type` الذي تم تعريفه من أجله |
| `<type as trait>::method(...)` | `Disambiguating` عن `Method call` عن طريق تسمية `Trait` و `type` |

يعرض الجدول B-4 `Symbols` التي تظهر في سياق استخدام (معلمات النوع العامة) `generic type parameters`.

<span class="caption">الجدول B-4: Generics</span>

| Symbol | الشرح |
| :--- | :--- |
| `path<...>` | يحدد (المعلمات) `parameters` لـ `generic type` في `type` (على سبيل المثال، `Vec<u8>`) |
| `path::<...>`, `method::<...>` | يحدد `parameters` لـ `generic type`، `function`، أو `method` في (تعبير) `expression`؛ يشار إليه غالبًا باسم _turbofish_ (على سبيل المثال، `"42".parse::<i32>()`) |
| `fn ident<...> ...` | تعريف `generic function` |
| `struct ident<...> ...` | تعريف (هيكل عام) `generic structure` |
| `enum ident<...> ...` | تعريف (تعداد عام) `generic enumeration` |
| `impl<...> ...` | تعريف (تنفيذ عام) `generic implementation` |
| `for<...> type` | (قيود العمر ذات الرتبة الأعلى) `Higher ranked lifetime bounds` |
| `type<ident=type>` | `Generic type` حيث تحتوي واحدة أو أكثر من `associated types` على (تعيينات محددة) `specific assignments` (على سبيل المثال، `Iterator<Item=T>`) |

يعرض الجدول B-5 `Symbols` التي تظهر في سياق تقييد `generic type parameters` باستخدام `trait bounds`.

<span class="caption">الجدول B-5: Trait Bound Constraints</span>

| Symbol | الشرح |
| :--- | :--- |
| `T: U` | `Generic parameter T` مقيد بـ `types` التي تنفذ `U` |
| `T: 'a` | `Generic type T` يجب أن يعيش أطول من `lifetime 'a` (مما يعني أن `type` لا يمكن أن يحتوي بشكل متعدٍ على أي (مراجع) `references` ذات `lifetimes` أقصر من `'a`) |
| `T: 'static` | `Generic type T` لا يحتوي على `borrowed references` بخلاف `static` |
| `'b: 'a` | `Generic lifetime 'b` يجب أن يعيش أطول من `lifetime 'a` |
| `T: ?Sized` | يسمح لـ `generic type parameter` بأن يكون (نوعًا بحجم ديناميكي) `dynamically sized type` |
| `'a + trait`, `trait + trait` | `Compound type constraint` |

يعرض الجدول B-6 `Symbols` التي تظهر في سياق استدعاء أو تعريف `Macros` وتحديد `Attributes` على `item`.

<span class="caption">الجدول B-6: Macros and Attributes</span>

| Symbol | الشرح |
| :--- | :--- |
| `#[meta]` | (سمة خارجية) `Outer attribute` |
| `#![meta]` | (سمة داخلية) `Inner attribute` |
| `$ident` | (استبدال الماكرو) `Macro substitution` |
| `$ident:kind` | (متغير الماكرو الوصفي) `Macro metavariable` |
| `$(...)...` | (تكرار الماكرو) `Macro repetition` |
| `ident!(...)`, `ident!{...}`, `ident![...]` | (استدعاء الماكرو) `Macro invocation` |

يعرض الجدول B-7 `Symbols` التي تنشئ `Comments`.

<span class="caption">الجدول B-7: Comments</span>

| Symbol | الشرح |
| :--- | :--- |
| `//` | (تعليق سطر) `Line comment` |
| `//!` | (تعليق توثيق سطر داخلي) `Inner line doc comment` |
| `///` | (تعليق توثيق سطر خارجي) `Outer line doc comment` |
| `/*...*/` | (تعليق كتلة) `Block comment` |
| `/*!...*/` | (تعليق توثيق كتلة داخلي) `Inner block doc comment` |
| `/**...*/` | (تعليق توثيق كتلة خارجي) `Outer block doc comment` |

يعرض الجدول B-8 السياقات التي تُستخدم فيها `Parentheses`.

<span class="caption">الجدول B-8: Parentheses</span>

| Symbol | الشرح |
| :--- | :--- |
| `()` | (مجموعة فارغة) `Empty tuple` (تُعرف أيضًا باسم `unit`)، حرفي ونوع |
| `(expr)` | (تعبير بين قوسين) `Parenthesized expression` |
| `(expr,)` | (تعبير مجموعة بعنصر واحد) `Single-element tuple expression` |
| `(type,)` | (نوع مجموعة بعنصر واحد) `Single-element tuple type` |
| `(expr, ...)` | (تعبير مجموعة) `Tuple expression` |
| `(type, ...)` | (نوع مجموعة) `Tuple type` |
| `expr(expr, ...)` | (تعبير استدعاء دالة) `Function call expression`؛ يستخدم أيضًا لتهيئة `tuple struct` و `tuple enum` |

يعرض الجدول B-9 السياقات التي تُستخدم فيها (الأقواس المعقوفة) `Curly Brackets`.

<span class="caption">الجدول B-9: Curly Brackets</span>

| السياق | الشرح |
| :--- | :--- |
| `{...}` | (تعبير كتلة) `Block expression` |
| `Type {...}` | (حرفي الهيكل) `Struct literal` |

يعرض الجدول B-10 السياقات التي تُستخدم فيها (الأقواس المربعة) `Square Brackets`.

<span class="caption">الجدول B-10: Square Brackets</span>

| السياق | الشرح |
| :--- | :--- |
| `[...]` | (حرفي المصفوفة) `Array literal` |
| `[expr; len]` | `Array literal` يحتوي على `len` نسخة من `expr` |
| `[type; len]` | (نوع المصفوفة) `Array type` يحتوي على `len` من `type` |
| `expr[expr]` | (فهرسة المجموعة) `Collection indexing`؛ `Overloadable` (`Index`, `IndexMut`) |
| `expr[..]`, `expr[a..]`, `expr[..b]`, `expr[a..b]` | `Collection indexing` يتظاهر بأنه (تقطيع المجموعة) `collection slicing`، باستخدام `Range`، `RangeFrom`، `RangeTo`، أو `RangeFull` كـ "index" |
