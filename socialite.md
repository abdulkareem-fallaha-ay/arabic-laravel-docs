# Laravel Socialite

- [مقدمة](#introduction)
- [الاثبيا](#installation)
- [تحديث Socialite](#upgrading-socialite)
- [الإعداد](#configuration)
- [المصادقة (Authentication)](#authentication)
    - [التوجيه (Routing)](#routing)
    - [المصادقة و التخزين](#authentication-and-storage)
    - [(Access Scopes) نطاقات الوصول](#access-scopes)
    - [بارامترات اختيارية](#optional-parameters)
- [استرجاع تفاصيل المستخدم](#retrieving-user-details)

<a name="introduction"></a>
## المقدمة

بالإضافة إلى المصادقة النموذجية القائمة على النموذج، يوفر Laravel أيضًا طريقة بسيطة, طريقة مريحة للمصادقة مع مزودي OAuth باستخدام [Laravel Socialite](https://github.com/laravel/socialite). تدعم Socialite حاليًا المصادقة مع Facebook و Twitter و LinkedIn و Google و GitHub و GitLab و Bitbucket.

> {نصيحة} يوجد محولات خاصة بالمواقع الأخرى تستطيع إيجادها عبر الموقع [Socialite Providers](https://socialiteproviders.com/).

<a name="installation"></a>
## التثبيت

للبدء مع Socialite، استخدم مدير حزمة Composer لإضافة الحزمة إلى تبعيات مشروعك:

```shell
composer require laravel/socialite
```

<a name="upgrading-socialite"></a>
## تحديث Socialite

عند الترقية إلى إصدار رئيسي جديد من Socialite، من المهم أن تراجع بعناية [دليل الترقية] (https://github.com/laravel/socialite/blob/master/UPGRADE.md).

<a name="configuration"></a>
## الإعداد

قبل استخدام Socialite، ستحتاج إلى إضافة بيانات اعتماد لمزودي OAuth الذين يستخدمهم تطبيقك. يجب وضع أوراق الاعتماد هذه في ملف التكوين الخاص بتطبيقك "config/services.php"، ويجب استخدام المفتاح "facebook" أو "twitter" أو "linkedin" أو "google" أو "github" أو "gitlab" أو "bitbuket'، اعتمادًا على مقدمي الخدمة التي يتطلبها:

    'github' => [
        'client_id' => env('GITHUB_CLIENT_ID'),
        'client_secret' => env('GITHUB_CLIENT_SECRET'),
        'redirect' => 'http://example.com/callback-url',
    ],

> {نصيحة} إذا كان خيار «إعادة التوجيه» يحتوي على مسار نسبي، فسيتم حله تلقائيًا إلى عنوان URL مؤهل تمامًا.

<a name="authentication"></a>
## المصادقة

<a name="routing"></a>
### Routing إعادة التوجيه

لمصادقة المستخدمين باستخدام مزود OAuth، ستحتاج إلى مسارين: أحدهما لإعادة توجيه المستخدم إلى مزود OAuth، والآخر لتلقي إعادة الاتصال من المزود بعد المصادقة. وتبين وحدة التحكم في المثال أدناه تنفيذ كلا المسارين:

    use Laravel\Socialite\Facades\Socialite;

    Route::get('/auth/redirect', function () {
        return Socialite::driver('github')->redirect();
    });

    Route::get('/auth/callback', function () {
        $user = Socialite::driver('github')->user();

        // $user->token
    });

طريقة «إعادة التوجيه» التي توفرها واجهة «Socialite» تعتني بإعادة توجيه المستخدم إلى مزود OAuth، بينما ستقرأ طريقة «المستخدم» الطلب الوارد وتستعيد معلومات المستخدم من المزود بعد المصادقة عليها.

<a name="authentication-and-storage"></a>
### المصادقة و التخزين

بمجرد استرداد المستخدم من مزود OAuth، يمكنك تحديد ما إذا    كان المستخدم موجودًا في قاعدة بيانات تطبيقك و [مصادقة المستخدم](/docs/{{version}}/authentication#authenticate-a-user-instance). إذا لم يكن المستخدم موجودًا في قاعدة بيانات تطبيقك، فستقوم عادةً بإنشاء سجل جديد في قاعدة بياناتك لتمثيل المستخدم:

    use App\Models\User;
    use Illuminate\Support\Facades\Auth;
    use Laravel\Socialite\Facades\Socialite;

    Route::get('/auth/callback', function () {
        $githubUser = Socialite::driver('github')->user();

        $user = User::where('github_id', $githubUser->id)->first();

        if ($user) {
            $user->update([
                'github_token' => $githubUser->token,
                'github_refresh_token' => $githubUser->refreshToken,
            ]);
        } else {
            $user = User::create([
                'name' => $githubUser->name,
                'email' => $githubUser->email,
                'github_id' => $githubUser->id,
                'github_token' => $githubUser->token,
                'github_refresh_token' => $githubUser->refreshToken,
            ]);
        }

        Auth::login($user);

        return redirect('/dashboard');
    });

> {نصيحة} لمزيد من المعلومات حول معلومات المستخدم المتاحة من مزودي OAuth المحددين، يرجى الرجوع إلى الوثائق على [استرجاع تفاصيل المستخدم](#retrieving-user-details).

<a name="access-scopes"></a>
### نطاقات الوصول

قبل إعادة توجيه المستخدم، يمكنك أيضًا إضافة «نطاقات» إضافية إلى طلب المصادقة باستخدام طريقة «النطاقات». ستدمج هذه الطريقة جميع النطاقات الموجودة مع النطاقات التي توفرها:

    use Laravel\Socialite\Facades\Socialite;

    return Socialite::driver('github')
        ->scopes(['read:user', 'public_repo'])
        ->redirect();

يمكنك كتابة جميع النطاقات الموجودة على طلب المصادقة باستخدام طريقة «set Scopes»:

    return Socialite::driver('github')
        ->setScopes(['read:user', 'public_repo'])
        ->redirect();

<a name="optional-parameters"></a>
### بارامترات اختيارية

يدعم عدد من مزودي OAuth المعلمات الاختيارية في طلب إعادة التوجيه. لإدراج أي معلمات اختيارية في الطلب، اتصل بطريقة «مع» مع مصفوفة ارتباطية:

    use Laravel\Socialite\Facades\Socialite;

    return Socialite::driver('google')
        ->with(['hd' => 'example.com'])
        ->redirect();

> {ملاحظة} عند استخدام طريقة «مع»، احرص على عدم اجتياز أي كلمات رئيسية محفوظة مثل «الحالة» أو «الاستجابة _ الكتابة».

<a name="retrieving-user-details"></a>
## استرجاع تفاصيل المستخدم

بعد إعادة توجيه المستخدم مرة أخرى إلى مسار إعادة الاتصال بالمصادقة، يمكنك استرداد تفاصيل المستخدم باستخدام طريقة «مستخدم» Socialite. يوفر كائن المستخدم الذي أعادته طريقة «المستخدم» مجموعة متنوعة من الخصائص والطرق التي يمكنك استخدامها لتخزين المعلومات حول المستخدم في قاعدة بياناتك الخاصة. قد تكون الخصائص والطرق المختلفة متاحة اعتمادًا على ما إذا كان مزود OAuth الذي تقوم بالمصادقة عليه يدعم OAuth 1.0 أو OAuth 2.0:

    use Laravel\Socialite\Facades\Socialite;

    Route::get('/auth/callback', function () {
        $user = Socialite::driver('github')->user();

        // OAuth 2.0 providers...
        $token = $user->token;
        $refreshToken = $user->refreshToken;
        $expiresIn = $user->expiresIn;

        // OAuth 1.0 providers...
        $token = $user->token;
        $tokenSecret = $user->tokenSecret;

        // All providers...
        $user->getId();
        $user->getNickname();
        $user->getName();
        $user->getEmail();
        $user->getAvatar();
    });

<a name="retrieving-user-details-from-a-token-oauth2"></a>
#### استرجاع تفاصيل المستخدم من الـ Token (OAuth2)

إذا كان لديك بالفعل رمز وصول صالح للمستخدم، فيمكنك استرداد تفاصيله باستخدام طريقة Socialite's «UserFromToken»:

    use Laravel\Socialite\Facades\Socialite;

    $user = Socialite::driver('github')->userFromToken($token);

<a name="retrieving-user-details-from-a-token-and-secret-oauth1"></a>
#### استرداد تفاصيل المستخدم من الـ Token  الـ Secret (OAuth1)

إذا كان لديك بالفعل رمز وسر صالح للمستخدم، فيمكنك استرداد تفاصيله باستخدام طريقة Socialite "المستخدم من TokenAndSecret':

    use Laravel\Socialite\Facades\Socialite;

    $user = Socialite::driver('twitter')->userFromTokenAndSecret($token, $secret);

<a name="stateless-authentication"></a>
#### Stateless مصادقة

يمكن استخدام طريقة «stateless» لتعطيل التحقق من الجلسة بناءً على state محددة. هذا مفيد عند إضافة المصادقة عبر المنصات الإجتماعية إلى API:

    use Laravel\Socialite\Facades\Socialite;

    return Socialite::driver('google')->stateless()->user();

> {ملاحظة} المصادقة عديمة الجنسية غير متاحة لـ Driver Twitter، الذي يستخدم OAuth 1.0 للمصادقة.
