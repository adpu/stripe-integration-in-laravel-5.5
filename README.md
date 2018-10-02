# Example of integrate Stripe payment gateway in Laravel 5.5

Steps:

1 -  Add this lines in require section of composer.json:


```
"cartalyst/stripe-laravel": "7.0.*"
```

2 - Run:

```
composer update
```

3 - Add this lines from your skype account inside .env file:

```
STRIPE_KEY=XXXXXXXXXX
STRIPE_SECRET=XXXXXXXXXX
```

4 - Inside config/app-php add 'provider' and 'aliases':

// Provider

```
Cartalyst\Stripe\Laravel\StripeServiceProvider::class,
```

// Aliases

```
'Stripe' => Cartalyst\Stripe\Laravel\Facades\Stripe::class,
```

5 - Inside routes/web.php add this lines

```
Route::get('addmoney/stripe', array('as' => 'addmoney.paywithstripe','uses' => 'AddMoneyController@payWithStripe'));
Route::post('addmoney/stripe', array('as' => 'addmoney.stripe','uses' => 'AddMoneyController@postPaymentWithStripe'));
```

6 - Create controller (AddMoneyController):

```
<?php
namespace App\Http\Controllers;
use App\Http\Requests;
use Illuminate\Http\Request;
use Validator;
use URL;
use Session;
use Redirect;
use Input;
use App\User;
use Stripe\Error\Card;
use Cartalyst\Stripe\Stripe;
class AddMoneyController extends Controller
{
 public function payWithStripe()
 {
 return view('paywithstripe');
 }
public function postPaymentWithStripe(Request $request)
 {
 $validator = Validator::make($request->all(), [
 'card_no' => 'required',
 'ccExpiryMonth' => 'required',
 'ccExpiryYear' => 'required',
 'cvvNumber' => 'required',
 'amount' => 'required',
 ]);
 $input = $request->all();
 if ($validator->passes()) { 
 $input = array_except($input,array('_token'));
 $stripe = Stripe::make('sk_test_U4n5mkkbFnqBTiCjtqvknMcm'); //set here your stripe secret key
 try {
 $token = $stripe->tokens()->create([
 'card' => [
 'number' => $request->get('card_no'),
 'exp_month' => $request->get('ccExpiryMonth'),
 'exp_year' => $request->get('ccExpiryYear'),
 'cvc' => $request->get('cvvNumber'),
 ],
 ]);
 // $token = $stripe->tokens()->create([
 // 'card' => [
 // 'number' => '4242424242424242',
 // 'exp_month' => 10,
 // 'cvc' => 314,
 // 'exp_year' => 2020,
 // ],
 // ]);
if (!isset($token['id'])) {
 return redirect()->route('addmoney.paywithstripe');
 }
 $charge = $stripe->charges()->create([
 'card' => $token['id'],
 'currency' => 'USD',
 'amount' => $request->get('amount'),
 'description' => 'Add in wallet',
 ]);
 
 if($charge['status'] == 'succeeded') {
 /**
 * Write Here Your Database insert logic.
 */
 echo "Pagament fet correctament";
 echo "<pre>";
 print_r($charge);exit();
 return redirect()->route('addmoney.paywithstripe');
 } else {
 \Session::put('error','Money not add in wallet!!');
 return redirect()->route('addmoney.paywithstripe');
 }
 } catch (Exception $e) {
 \Session::put('error',$e->getMessage());
 return redirect()->route('addmoney.paywithstripe');
 } catch(\Cartalyst\Stripe\Exception\CardErrorException $e) {
 \Session::put('error',$e->getMessage());
 return redirect()->route('addmoney.paywithstripe');
 } catch(\Cartalyst\Stripe\Exception\MissingParameterException $e) {
 \Session::put('error',$e->getMessage());
 return redirect()->route('addmoney.paywithstripe');
 }
 }
 }
}
```


7 - Create view (paywithstripe.blade.php)

```
<html>
<head>
    </head>
    <body>

 <form class="form-horizontal" method="POST" id="payment-form" role="form" action="{!!route('addmoney.stripe')!!}" >
{{ csrf_field() }}
 
 
 <label class='control-label'>Card Number</label>
 <input autocomplete='off' class='form-control card-number' size='20' type='text' name="card_no">
 
 <label class='control-label'>CVV</label>
 <input autocomplete='off' class='form-control card-cvc' placeholder='ex. 311' size='4' type='text' name="cvvNumber">
 
 <label class='control-label'>Expiration</label>
 <input class='form-control card-expiry-month' placeholder='MM' size='2' type='text' name="ccExpiryMonth">
 
 <label class='control-label'> </label>
 <input class='form-control card-expiry-year' placeholder='YYYY' size='4' type='text' name="ccExpiryYear">
 <input class='form-control card-expiry-year' placeholder='YYYY' size='4' type='hidden' name="amount" value="300">
 
 Total:
 <span class='amount'>$300</span>
 
 <button class='form-control btn btn-primary submit-button' type='submit'>Pay »</button>
 
 <div class='error hide'>
 <div class='alert-danger alert'>
 Please correct the errors and try again.
 </div>
 </div>
 
 </form>


</body>
</html>
```


8 - Test it calling:  http://mydomain.com/addmoney/stripe

# References:

https://medium.com/@kshitij206/how-to-integrate-stripe-payment-gateway-in-laravel-5-5-c2965854b983
