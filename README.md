## INSTALLATION ##

1. Copy Charge folder to `site/addons`
2. Create (or copy) the settings file to `site/settings/addons/charge.yaml`
3. Run `php please addons:refresh`

## SETTINGS ##

* in `charge.yaml`
    * `formset` - which formset you'll be using for the Form submission
    * `currency` - default currency for payments
    * `offer_memberships` - if you offer membership (on user registration), set this to `true`
* in your `.env` file, which MUST NOT be checked in:
	* `STRIPE_SECRET_KEY` - your stripe secret key, found here: https://dashboard.stripe.com/account/apikeys
	* `STRIPE_PUBLIC_KEY` - your stripe public key, found here: https://dashboard.stripe.com/account/apikeys

## USAGE ##

*NOTE*: all ways below require `{{ charge:js }}` be loaded on the appropriate template. I recommend using the [yield](https://docs.statamic.com/tags/yield) and [section](https://docs.statamic.com/tags/section) tags for that.

A Stripe Customer is created on a charge, unless the customer has been charged before (via Charge).

There are three ways to use it
1. Statamic's `Form` tag
2. Charge's `{{ charge:form }}`- for when you want to use Stripe Checkout, etc
3. User registration form (for paid memberships, both subscriptions and one-time)

Charge Form

* for a one-time charge pass in the `amount`, `description`, and optionally the `currency` as parameters on the tag
* for a subscription, have a `plan` field in your form with the Stripe Plan
* if you want to redirect the customer after the charge, use a `redirect` parameter
* `{{ success }}` and `{{ details }}` are available to you after a successful charge.

Statamic Form

* the following fields *must* be in your form:
    * `stripeEmail` - email of customer
* for a one-time charge, somewhere in your form you need to set the `amount`, `description`, and optionally `currency` via `{{ charge:data }}`
* for a subscription, include a `plan` field along with the above email field. Currency, amount nor description are needed for subscriptions
* the `customer_id` is available in the `submission` data
* please note the `data-*` attributes on the form items. Those are required.

Example - Charge Form - Stripe Checkout:
```
{{# currency is optional #}}
{{ charge:form redirect="/thanks" amount="{amount}" description="{description}" currency="usd" }}
    {{ if success }}
        {{ details }}
            ID: {{ id }}
        {{ /details }}
    {{ /if}}

    <script
            src="https://checkout.stripe.com/checkout.js" class="stripe-button"
            data-key="{{ env: STRIPE_PUBLIC_KEY }}"
            data-amount="{{ amount }}"
            data-name="{{ company }}"
            data-description="{{ description }}"
            data-image="/img/documentation/checkout/marketplace.png"
            data-locale="auto"
            data-currency="{{ currency }}">
    </script>
{{ /charge:form }}
```

Example - Statamic Form:
```
{{ form:create in="charge" attr="class:form|data-charge-form" redirect="/thanks" }}
	<div class="form-item">
		<label>Email</label>
		<input type="email" name="stripeEmail" value="{{ old:stripeEmail }}" />
	</div>


	<fieldset class="payment">
		<div class="form-item">
			<label for="cc_name">Name on Card</label>
			<input type="text" data-stripe="name" id="cc_name" required>
		</div>

		<div class="form-item">
			<label for="cc_number">Zip/Postal Code</label>
			<input type="text" data-stripe="number" id="address_zip" required>
		</div>

		<div class="form-item">
			<label for="cc_number">Card Number</label>
			<input type="text" data-stripe="number" id="cc_number" required>
		</div>

		<div class="row row-inner">
			<div class="col">
				<div class="form-item">
					<label for="exp_month">Expiry Month</label>
					<input type="text" data-stripe="exp_month" id="exp_month" maxlength="2" required placeholder="00">
				</div>
			</div>

			<div class="col">
				<div class="form-item">
					<label for="exp_year">Expiry Year</label>
					<input type="text" data-stripe="exp_year" id="exp_year" maxlength="2" required placeholder="00">
				</div>
			</div>

			<div class="col">
				<div class="form-item">
					<label for="cvc">CVC</label>
					<input type="text" data-stripe="cvc" id="cvc" maxlength="3" required placeholder="000">
				</div>
			</div>
		</div>

	</fieldset>

	{{ charge:data amount="{amount}" description="{description}" currency="cad" }}

	<button class="button primary" id="register" data-charge-button>Pay</button>

{{ /form:create }}
```

For a subscription, like above, but no `{{ charge:data }}` needed, instead:
```
                    <div class="form-item">
                        <label for="plan">Membership Type</label>
                        <select name="plan" id="plan" class="big" >
                            <option>Please Select</option>
                            <option value="associate">Associate</option>
                            <option value="clinical">Clinical</option>
                            <option value="student">Student</option>
                        </select>
                    </div>
```

For a membership upon user registration:
```
<section class="regular">

    <header>
        <h1>Register</h1>
    </header>

    <article class="content">

        {{ user:register_form redirect="/account" attr="class:form|data-charge-form" }}

            {{ if errors }}
                <div class="alert alert-danger">
                    {{ errors }}
                        {{ value }}<br>
                    {{ /errors }}
                </div>
            {{ /if }}

        <div class="row row-inner">
            <div class="col">
                <div class="form-item">
                    <label>Username</label>
                    <input type="text" name="username" value="{{ old:username }}" />
                </div>
            </div>

            <div class="col">
                <div class="form-item">
                    <label>First Name</label>
                    <input type="text" name="first_name" value="{{ old:first_name }}" />
                </div>
            </div>

            <div class="col">
                <div class="form-item">
                    <label>Last Name</label>
                    <input type="text" name="last_name" value="{{ old:last_name }}" />
                </div>
            </div>
        </div>

        <div class="form-item">
            <label>Email</label>
            <input type="email" name="email" value="{{ old:email }}" />
        </div>

        <div class="form-item">
            <label>Password</label>
            <input type="password" name="password" />
        </div>

        <div class="form-item">
            <label>Password Confirmation</label>
            <input type="password" name="password_confirmation" />
        </div>

        <fieldset class="payment">
            <legend>Payment</legend>
            <div class="form-item">
                <label for="plan">Membership Type</label>
                <select name="plan" id="plan" class="big" >
                    <option>Please Select</option>
                    <option value="associate">Associate</option>
                    <option value="clinical">Clinical</option>
                    <option value="student">Student</option>
                </select>
            </div>

            <div class="form-item">
                <label for="cc_name">Name on Card</label>
                <input type="text" data-stripe="name" id="cc_name" required>
            </div>

            <div class="form-item">
                <label for="cc_number">Zip/Postal Code</label>
                <input type="text" data-stripe="number" id="address_zip" required>
            </div>

            <div class="form-item">
                <label for="cc_number">Card Number</label>
                <input type="text" data-stripe="number" id="cc_number" required>
            </div>

            <div class="row row-inner">
                <div class="col">
                    <div class="form-item">
                        <label for="exp_month">Expiry Month</label>
                        <input type="text" data-stripe="exp_month" id="exp_month" maxlength="2" required placeholder="00">
                    </div>
                </div>

                <div class="col">
                    <div class="form-item">
                        <label for="exp_year">Expiry Year</label>
                        <input type="text" data-stripe="exp_year" id="exp_year" maxlength="2" required placeholder="00">
                    </div>
                </div>

                <div class="col">
                    <div class="form-item">
                        <label for="cvc">CVC</label>
                        <input type="text" data-stripe="cvc" id="cvc" maxlength="3" required placeholder="000">
                    </div>
                </div>
            </div>

        </fieldset>

        <button class="button primary" id="register" data-charge-button>Register</button>

        {{ /user:register_form }}

    </article>

</section>
{{ section:chargeJS }}
    {{ charge:js }}
{{ /section:chargeJS }}
```

For a one-time, take out the `plan` part and use `{{ charge:data }}` for the amount, etc