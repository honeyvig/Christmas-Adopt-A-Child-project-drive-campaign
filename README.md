# Christmas-Adopt-A-Child-project-drive-campaign
Dear Santa Helper website with integration of the Friends of Parkside's (FOP) First Annual Campaign Drive, we can break the task down into multiple steps. Since the scope includes both front-end and back-end changes, we’ll focus on a Python web framework (e.g., Flask or Django) for the backend, with some HTML, CSS, and JavaScript for the front-end updates.
Overview of the Steps:

    Set up the Backend (Flask/Django) to handle the different sections for Dear Santa Helper and Campaign Drive.
    Integrate Donation Functionality: Using a payment gateway like Stripe or PayPal for donations.
    Design and Layout Improvements: Update the website’s design to reflect a cohesive experience for both initiatives.
    Mobile Responsiveness: Make sure the site is fully responsive.
    Content Development and SEO: Enhance content for SEO and user engagement.

Assumptions:

    We’re using Flask for simplicity and flexibility.
    We'll use Stripe for donation handling (this can be replaced with PayPal or another service if necessary).
    Frontend updates will utilize Bootstrap for responsive design.

Step 1: Install Dependencies

Make sure you have Flask and other necessary libraries installed. You can also use Stripe if handling donations via credit card.

pip install Flask Stripe

Step 2: Setup Flask Application

Create a basic Flask application structure:

/DearSantaHelper
    /static
        /css
            style.css
    /templates
        index.html
        donate.html
    app.py
    requirements.txt

requirements.txt:

Flask
Stripe

Step 3: Flask Backend (app.py)

from flask import Flask, render_template, request, redirect, url_for
import stripe

app = Flask(__name__)

# Set your Stripe keys (ensure you're using live keys in production)
stripe.api_key = 'sk_test_your_secret_key'

# Route for the home page
@app.route('/')
def index():
    return render_template('index.html')

# Route for the First Annual Campaign Drive section
@app.route('/campaign')
def campaign():
    return render_template('campaign.html')

# Route for the Dear Santa Helper Project section
@app.route('/santa_helper')
def santa_helper():
    return render_template('santa_helper.html')

# Donation page route
@app.route('/donate', methods=['GET', 'POST'])
def donate():
    if request.method == 'POST':
        amount = request.form['amount']
        donation_type = request.form['donation_type']

        # Convert amount to the correct format (Stripe expects cents)
        amount_in_cents = int(float(amount) * 100)

        try:
            # Create a new Stripe Payment Intent
            intent = stripe.PaymentIntent.create(
                amount=amount_in_cents,
                currency='usd',
                description=f'Donation to {donation_type}'
            )
            return render_template('donate.html', client_secret=intent.client_secret)
        except Exception as e:
            return str(e)
    
    return render_template('donate.html')

# Route to handle the payment success
@app.route('/success')
def success():
    return render_template('success.html')

# Route to handle the payment cancellation
@app.route('/cancel')
def cancel():
    return render_template('cancel.html')

if __name__ == '__main__':
    app.run(debug=True)

Step 4: Frontend HTML Templates
1. index.html (Main Landing Page with Links to Projects and Campaign)

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dear Santa Helper</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="{{ url_for('static', filename='css/style.css') }}">
</head>
<body>
    <div class="container">
        <h1 class="text-center mt-5">Welcome to Dear Santa Helper</h1>
        <div class="row mt-5">
            <div class="col-md-6">
                <h3>About the Project</h3>
                <p>Dear Santa Helper helps children in need with holiday gifts and year-round support. Learn more about the campaign and get involved.</p>
                <a href="/santa_helper" class="btn btn-primary">Learn About the Project</a>
            </div>
            <div class="col-md-6">
                <h3>First Annual Campaign Drive</h3>
                <p>Friends of Parkside (FOP) is hosting their first annual campaign drive to support community programs.</p>
                <a href="/campaign" class="btn btn-primary">Learn About the Campaign</a>
            </div>
        </div>
    </div>
</body>
</html>

2. campaign.html (Campaign Information Page)

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>First Annual Campaign Drive</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <div class="container">
        <h1 class="text-center mt-5">Friends of Parkside - First Annual Campaign Drive</h1>
        <p>This campaign is dedicated to raising funds for community programs that benefit children and families. Your contributions will help us provide resources and support throughout the year.</p>
        <h3>How You Can Help</h3>
        <ul>
            <li>Donate online</li>
            <li>Volunteer for events</li>
            <li>Spread the word</li>
        </ul>
        <a href="/donate" class="btn btn-success">Donate Now</a>
    </div>
</body>
</html>

3. donate.html (Donation Page)

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Donate to Dear Santa Helper</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <div class="container">
        <h1 class="text-center mt-5">Support Our Mission</h1>
        <form action="/donate" method="post">
            <div class="mb-3">
                <label for="amount" class="form-label">Donation Amount ($)</label>
                <input type="text" class="form-control" id="amount" name="amount" required>
            </div>
            <div class="mb-3">
                <label for="donation_type" class="form-label">Select Donation Type</label>
                <select class="form-control" id="donation_type" name="donation_type">
                    <option value="Dear Santa Helper">Dear Santa Helper</option>
                    <option value="Campaign Drive">Campaign Drive</option>
                </select>
            </div>
            <button type="submit" class="btn btn-primary">Donate</button>
        </form>

        {% if client_secret %}
        <h3>Complete your donation</h3>
        <p>Click below to securely complete your payment via Stripe.</p>
        <script src="https://js.stripe.com/v3/"></script>
        <script>
            var stripe = Stripe('pk_test_your_public_key');
            var clientSecret = '{{ client_secret }}';

            stripe.confirmCardPayment(clientSecret, {
                payment_method: {
                    card: {
                        number: '4111111111111111',
                        exp_month: '12',
                        exp_year: '2023',
                        cvc: '123'
                    }
                }
            }).then(function(result) {
                if (result.error) {
                    alert("Error: " + result.error.message);
                } else {
                    window.location.href = "/success";
                }
            });
        </script>
        {% endif %}
    </div>
</body>
</html>

Step 5: Mobile Responsiveness with Bootstrap and Custom CSS

To ensure the website is mobile-friendly, you can rely on Bootstrap's grid system and responsiveness. Customize the design further using custom CSS in the style.css file.

/* /static/css/style.css */
body {
    font-family: Arial, sans-serif;
}

h1, h3 {
    color: #333;
}

.btn {
    margin-top: 20px;
}

Step 6: Test and Launch

    Test Functionality: Run the Flask application locally and test all routes, donation functionality, and mobile responsiveness.
    Deploy: Once testing is complete, deploy the app using a service like Heroku, AWS, or DigitalOcean.

Final Notes:

    The provided solution includes the creation of a donation section, campaign page, and a refreshed design using Flask.
    For a real-world deployment, integrate Stripe securely for payments, and ensure the content is optimized for SEO and mobile users.
    Make sure to use environment variables to securely store keys and credentials.
