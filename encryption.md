# التشفير

- [مقدمة](#مقدمة)
- [الاعداد](#الاعداد)
- [استخدام التشفير](#استخدام-التشفير)

<a name="introduction"></a>
## مقدمة

توفر خدمات التشفير في Laravel واجهة بسيطة ومريحة لتشفير وفك تشفير النص عبر OpenSSL باستخدام تشفير AES-256 و AES-128. يتم توقيع جميع قيم Laravel المشفرة باستخدام رمز مصادقة الرسالة (MAC) بحيث لا يمكن تعديل قيمتها الأساسية أو العبث بها بمجرد تشفيرها.

<a name="configuration"></a>
## الضبط والاعداد

قبل البدء باستخدام التشفير في Laravel, يجب عليك اولاً ضبط قيمة المفتاح `key` ضمن ملف التكوين الخاص بك أي ضمن `config/app.php`. هذه القيمة تعتمد على قيمة متغيير البيئة `APP_KEY`. يجب عليك استخدام الأمر `php artisan key:generate` حتى تولّد قيمة متغيّر البيئة هذا حيث أن الأمر `key:generate` سيستخدم منشئ البايت العشوائي الآمن (secure random bytes generator) الخاص بـ PHP لإنشاء مفتاح آمن مشفر لتطبيقك. عادةً, ما يتم توليد قيمة متغيّر البيئة `APP_KEY` الخاص بك خلال مرحلة [تثبيت Laravel]  (/docs/{{version}}/installation). 

<a name="using-the-encrypter"></a>
## استخدام التشفير

<a name="encrypting-a-value"></a>
#### تشفير قيمة ما

يمكنك تشفير قيمة ما باستخدام طريقة `encryptString` التي توفرها الواجهة `Crypt`. يتم تشفير جميع القيم المراد تشفيرها باستخدام تشفير OpenSSL و AES-256-CBC. علاوة على ذلك ، يتم توقيع جميع القيم المشفرة برمز مصادقة الرسالة (MAC). سيمنع رمز مصادقة الرسالة المتكاملة فك تشفير أي قيمة تم العبث بها من قبل المستخدمين الضارين (أي سوف يمنع التلاعب بالبيانات ويضمن التكامل):

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Models\User;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Crypt;

    class DigitalOceanTokenController extends Controller
    {
        /**
         * Store a DigitalOcean API token for the user.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return \Illuminate\Http\Response
         */
        public function storeSecret(Request $request)
        {
            $request->user()->fill([
                'token' => Crypt::encryptString($request->token),
            ])->save();
        }
    }

<a name="decrypting-a-value"></a>
#### فك تشفير قيمة ما

يمكنك فك تشفير قيمة ما باستخدام طريقة `decryptString` التي توفرها الواجهة` Crypt`. إذا تعذر فك تشفير القيمة بشكل صحيح ، كما هو الحال عندما يكون رمز مصادقة الرسالة غير صالح ، فسيتم اطلاق الاستثناء التالي `Illuminate \ Contracts \ Encryption \ DecryptException`:

    use Illuminate\Contracts\Encryption\DecryptException;
    use Illuminate\Support\Facades\Crypt;

    try {
        $decrypted = Crypt::decryptString($encryptedValue);
    } catch (DecryptException $e) {
        //
    }
