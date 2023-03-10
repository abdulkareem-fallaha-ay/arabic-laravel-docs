# إنشاء عنوان URL

- [المقدمة](#introduction)
- [الأساسيات](#the-basics)
    - [إنشاء عناوين URLs](#generating-urls)
    - [الوصول إلى عنوان URL الحالي](#accessing-the-current-url)
- [عناوين URLs لمسار ذو اسم](#urls-for-named-routes)
    - [عناوين URLs ذات توقيع](#signed-urls)
- [عناوين URLs لتوابع متحكم](#urls-for-controller-actions)
- [القيم الافتراضية](#default-values)

<a name="introduction"></a>
## المقدمة

يوفر Laravel العديد من المُساعدات لمساعدتك في إنشاء عناوين URL لتطبيقك. هؤلاء المُساعدات تفيد بشكل أساسي عند إنشاء روابط في القوالب واستجابات API ، أو عند إنشاء استجابات إعادة التوجيه إلى جزء آخر من تطبيقك.

<a name="the-basics"></a>
## الأساسيات

<a name="generating-urls"></a>
### إنشاء عناوين URLs

يمكن استخدام المساعد `url` لإنشاء عناوين URL عشوائية لتطبيقك. العنوان URL المنشئ سيستخدم بشكل تلقائي النظام (HTTP أو HTTPS) والمضيف (host) من الطلب الحالي الذي يتعامل معه التطبيق:

    $post = App\Models\Post::find(1);

    echo url("/posts/{$post->id}");

    // http://example.com/posts/1

<a name="accessing-the-current-url"></a>
### الوصول إلى عنوان URL الحالي

إذا لم يتم توفير مسار إلى المُساعد `url` ، فسيتم إرجاع نموذج ` Illuminate \ Routing \ UrlGenerator` ، مما يسمح لك بالوصول إلى معلومات حول عنوان URL الحالي:

    // الحصول على عنوان الحالي بدون الاستعلام النصي ...
    echo url()->current();

    // الحصول على عنوان الحالي متضمناً الاستعلام النصي ...
    echo url()->full();

    // الحصول على عنوان الكامل للطلب السابق ...
    echo url()->previous();

يمكن أيضاً الوصول إلى كل طريقة من هذه الطرق عبر `URL` [facade](/docs/{{version}}/facades)

    use Illuminate\Support\Facades\URL;

    echo URL::current();

<a name="urls-for-named-routes"></a>
## عناوين URLs لمسار ذو اسم

يمكن استخدام المُساعد `route` لإنشاء عناوين URL إلى [المسارات المسماة] (/docs/{{version}}/routing#named-routes). تتيح لك المسارات المسماة إنشاء عناوين URL دون أن تقترن بعنوان URL الفعلي المحدد في المسار. لذلك ، إذا تغير عنوان URL للمسار ، فلا داعي لإجراء تغييرات على طلباتك على التابع `route`. على سبيل المثال ، تخيل أن التطبيق الخاص بك يحتوي على مسار محدد كما يلي:

    Route::get('/post/{post}', function (Post $post) {
        //
    })->name('post.show');

لإنشاء عنوان URL لهذا المسار ، يمكنك استخدام المُساعد `route` كالتالي:

    echo route('post.show', ['post' => 1]);

    // http://example.com/post/1

بالطبع ، يمكن أيضًا استخدام المُساعد `route` لإنشاء عناوين URL لمسارات ذات مدخلات متعددة:

    Route::get('/post/{post}/comment/{comment}', function (Post $post, Comment $comment) {
        //
    })->name('comment.show');

    echo route('comment.show', ['post' => 1, 'comment' => 3]);

    // http://example.com/post/1/comment/3

ستتم إضافة أي عناصر مصفوفة إضافية لا تتوافق مع المدخلات المعرفة للمسار إلى الاستعلام النصي للعنوان URL:

    echo route('post.show', ['post' => 1, 'search' => 'rocket']);

    // http://example.com/post/1?search=rocket

<a name="eloquent-models"></a>
#### نماذج Eloquent

ستنشئ غالبًا عناوين URL باستخدام مفتاح المسار (عادةً المفتاح الأساسي) من [Eloquent نماذج] (/docs/{{version}}/eloquent). لهذا السبب ، يمكنك تمرير نماذج Eloquent كقيم للمدخلات. وسيقوم المُساعد `route` باستخراج مفتاح مسار النموذج تلقائيًا:

    echo route('post.show', ['post' => $post]);

<a name="signed-urls"></a>
### عناوين URLs ذات توقيع

تُتيح لك لارافل إنشاء عناوين URL "ذات توقيع" بسهولة للمسارات المسماة. تحتوي هذه العناوين URL هذه على "توقيع" مشفر ملحق بالاستعلام النصي والتي تسمح للارافل بالتحقق من أن عنوان URL لم يتم تعديله منذ أن تم إنشائه. تعد عناوين URL ذات توقيع مفيدة بشكل خاص للمسارات التي يمكن الوصول إليها بشكل عام ولكنها تحتاج إلى طبقة من الحماية ضد التلاعب بعنوان URL.

على سبيل المثال ، يمكنك استخدام عناوين URL ذات توقيع لتنفيذ رابط "إلغاء اشتراك" عام يتم إرساله بالبريد الإلكتروني إلى عملائك. لإنشاء عنوان URL موقّع لمسار محدد ، استخدم الطريقة `signatureRoute` للواجهة` URL`:

    use Illuminate\Support\Facades\URL;

    return URL::signedRoute('unsubscribe', ['user' => 1]);

إذا كنت ترغب في إنشاء عنوان URL مؤقت ذو توقيع ينتهي صلاحيته بعد فترة زمنية محددة ، يمكنك استخدام طريقة `temporarySignedRoute`. عندما تتحقق لارافل من صحة عنوان URL مؤقت ذو توقيع، فإنها ستضمن عدم انقضاء الوقت الزمني لانتهاء الصلاحية المشفر في عنوان URL الموقع:

    use Illuminate\Support\Facades\URL;

    return URL::temporarySignedRoute(
        'unsubscribe', now()->addMinutes(30), ['user' => 1]
    );

<a name="validating-signed-route-requests"></a>
#### Validating Signed Route Requests

To verify that an incoming request has a valid signature, you should call the `hasValidSignature` method on the incoming `Illuminate\Http\Request` instance:

    use Illuminate\Http\Request;

    Route::get('/unsubscribe/{user}', function (Request $request) {
        if (! $request->hasValidSignature()) {
            abort(401);
        }

        // ...
    })->name('unsubscribe');

Sometimes, you may need to allow your application's frontend to append data to a signed URL, such as when performing client-side pagination. Therefore, you can specify request query parameters that should be ignored when validating a signed URL using the `hasValidSignatureWhileIgnoring` method. Remember, ignoring parameters allows anyone to modify those parameters on the request:

    if (! $request->hasValidSignatureWhileIgnoring(['page', 'order'])) {
        abort(401);
    }

Instead of validating signed URLs using the incoming request instance, you may assign the `Illuminate\Routing\Middleware\ValidateSignature` [middleware](/docs/{{version}}/middleware) to the route. If it is not already present, you should assign this middleware a key in your HTTP kernel's `routeMiddleware` array:

    /**
     * The application's route middleware.
     *
     * These middleware may be assigned to groups or used individually.
     *
     * @var array
     */
    protected $routeMiddleware = [
        'signed' => \Illuminate\Routing\Middleware\ValidateSignature::class,
    ];

Once you have registered the middleware in your kernel, you may attach it to a route. If the incoming request does not have a valid signature, the middleware will automatically return a `403` HTTP response:

    Route::post('/unsubscribe/{user}', function (Request $request) {
        // ...
    })->name('unsubscribe')->middleware('signed');

<a name="responding-to-invalid-signed-routes"></a>
#### Responding To Invalid Signed Routes

When someone visits a signed URL that has expired, they will receive a generic error page for the `403` HTTP status code. However, you can customize this behavior by defining a custom "renderable" closure for the `InvalidSignatureException` exception in your exception handler. This closure should return an HTTP response:

    use Illuminate\Routing\Exceptions\InvalidSignatureException;

    /**
     * Register the exception handling callbacks for the application.
     *
     * @return void
     */
    public function register()
    {
        $this->renderable(function (InvalidSignatureException $e) {
            return response()->view('error.link-expired', [], 403);
        });
    }

<a name="urls-for-controller-actions"></a>
## URLs For Controller Actions

The `action` function generates a URL for the given controller action:

    use App\Http\Controllers\HomeController;

    $url = action([HomeController::class, 'index']);

If the controller method accepts route parameters, you may pass an associative array of route parameters as the second argument to the function:

    $url = action([UserController::class, 'profile'], ['id' => 1]);

<a name="default-values"></a>
## Default Values

For some applications, you may wish to specify request-wide default values for certain URL parameters. For example, imagine many of your routes define a `{locale}` parameter:

    Route::get('/{locale}/posts', function () {
        //
    })->name('post.index');

It is cumbersome to always pass the `locale` every time you call the `route` helper. So, you may use the `URL::defaults` method to define a default value for this parameter that will always be applied during the current request. You may wish to call this method from a [route middleware](/docs/{{version}}/middleware#assigning-middleware-to-routes) so that you have access to the current request:

    <?php

    namespace App\Http\Middleware;

    use Closure;
    use Illuminate\Support\Facades\URL;

    class SetDefaultLocaleForUrls
    {
        /**
         * Handle the incoming request.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return \Illuminate\Http\Response
         */
        public function handle($request, Closure $next)
        {
            URL::defaults(['locale' => $request->user()->locale]);

            return $next($request);
        }
    }

Once the default value for the `locale` parameter has been set, you are no longer required to pass its value when generating URLs via the `route` helper.

<a name="url-defaults-middleware-priority"></a>
#### URL Defaults & Middleware Priority

Setting URL default values can interfere with Laravel's handling of implicit model bindings. Therefore, you should [prioritize your middleware](/docs/{{version}}/middleware#sorting-middleware) that set URL defaults to be executed before Laravel's own `SubstituteBindings` middleware. You can accomplish this by making sure your middleware occurs before the `SubstituteBindings` middleware within the `$middlewarePriority` property of your application's HTTP kernel.

The `$middlewarePriority` property is defined in the base `Illuminate\Foundation\Http\Kernel` class. You may copy its definition from that class and overwrite it in your application's HTTP kernel in order to modify it:

    /**
     * The priority-sorted list of middleware.
     *
     * This forces non-global middleware to always be in the given order.
     *
     * @var array
     */
    protected $middlewarePriority = [
        // ...
         \App\Http\Middleware\SetDefaultLocaleForUrls::class,
         \Illuminate\Routing\Middleware\SubstituteBindings::class,
         // ...
    ];
