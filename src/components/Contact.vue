<template>
  <h1 class="text-center my-4 pt-5" id="contact">Contact</h1>
    <div class="contact-section">
        <div class="row align-items-center mt-4">
            <div class="col-md-6 map-container">
                <iframe id="gmap_canvas" src="https://maps.google.com/maps?q=centro%20escolar%20university%20manila&t=&z=13&ie=UTF8&iwloc=&output=embed" frameborder="0" scrolling="no" marginheight="0" marginwidth="0"></iframe>
            </div>
            <div class="col-md-6">
                <!-- binds the submitForm() function to the form submit event with @submit.prevent -->
                <form @submit.prevent="submitForm">
                    <div class="mb-3">
                        <!-- bind the "name" state to the form input v-model -->
                        <input type="text" v-model="name" class="form-control contact-form-control" placeholder="First Name M.I. Last Name">
                    </div>
                    <div class="mb-3">
                        <input type="email" v-model="email" class="form-control contact-form-control" placeholder="Email">
                    </div>
                    <div class="mb-3">
                        <textarea v-model="message" class="form-control contact-form-control" rows="6" placeholder="Message"></textarea>
                    </div>
                    <div class="form-footer">
                        <div class="social-icons">
<!--                                <a href="https://www.facebook.com/profile.php?id=100085701498879" id="facebook"><i class="fab fa-facebook"></i></a> -->
                            <a href="https://www.linkedin.com/in/charles-babbage-8291a6211/" id="linkedin"><i class="fab fa-linkedin"></i></a>
                            <a href="https://gitlab.com/cbabbage0991" id="gitlab"><i class="fab fa-gitlab"></i></a>
                            <a href="https://github.com/cbabbage0991" id="github"><i class="fab fa-github"></i></a>
                        </div>

                        <!-- bind isLoading state to the form button with the :disabled attribute to disable the button when sending the form. -->
                        <button type="submit" class="submit-btn pl-5 pr-5" :disabled="isLoading">
                            {{isLoading ? "Sending..." : "Submit" }}
                        </button>

                        <!-- recaptcha checkbox -->
                        <div class="d-flex justify-content-end mt-2">
                            <div ref="recaptchaContainer"></div>
                        </div>

                    </div>
                </form>
                
            </div>
        </div>
    </div>
</template>




<script setup>

import { ref, onMounted, onBeforeMount } from "vue";


import { Notyf } from "notyf";
import 'notyf/notyf.min.css';


const notyf = new Notyf();



// Add your web3forms access key here
// We can also store this in our our .env file.
const WEB3FORMS_ACCESS_KEY = "92c148aa-5e00-45a2-ab1e-f1f6a6e84be0";


// This will be the subject template of the email you will receive from web3forms when someone submits the contact form.
const subject = "New message from Portfolio Contact Form";


// Initial values for the form inputs
// We use of the ref() to make them reactive.
const name = ref("");
const email = ref("");
const message = ref("");

// To simulate the loading state in our front end app.
const isLoading = ref(false);




const submitForm = async () => {

    // Checks if reCaptcha token is present, returns an error when not verified.
    if (!recaptchaToken.value) {
        notyf.error('Please verify that you are not a robot.');
        return;
    }


    // Set the loading state to true
    isLoading.value = true;

    try {

        const response = await fetch("https://api.web3forms.com/submit", {
            method: "POST",
            headers: {
                "Content-Type": "application/json",
                Accept: "application/json",
            },
            body: JSON.stringify({
                access_key: WEB3FORMS_ACCESS_KEY,
                subject: subject,
                name: name.value,
                email: email.value,
                message: message.value,
            }),

        });

        const result = await response.json();

        // Checks if the form submission was successful.
            //  If yes, displays the notification toast.
        if(result.success) {
            console.log(result);

            isLoading.value = false;
            notyf.success("Message Sent!");


            // Clear form fields
            name.value = "";
            email.value = "";
            message.value = "";

        }

    } catch (error) {
        // If not successful, display failure notification toast.
        console.log(error);

        isLoading.value = false;
        notyf.error("Failed to send message. Please try again.");

    } finally {

        // Reset captcha after submit or error.
        resetRecaptcha();

    }


}



const SITE_KEY = "6LfjsHEsAAAAANNa0dgWXsdiZX4NotmtM_B0eTMG"

const recaptchaContainer = ref(null);
const recaptchaWidgetId = ref(null);
const recaptchaToken = ref('');


// Once successfully verified the token will be stored in the recaptchaToken state variable.
function onRecaptchaSuccess(token) {
  recaptchaToken.value = token;
}

// Once the has been expired or is invalid.
function onRecaptchaExpired() {
  recaptchaToken.value = '';
}

// Renders the recaptcha component in our app.
function renderRecaptcha() {
  if (!window.grecaptcha) {
    console.error('reCAPTCHA not loaded');
    return;
  }

  recaptchaWidgetId.value = window.grecaptcha.render(recaptchaContainer.value, {
    sitekey: SITE_KEY,
    size: 'normal', // or 'compact'
    callback: onRecaptchaSuccess,
    'expired-callback': onRecaptchaExpired,
  });
}

// Function to reset reCaptcha
function resetRecaptcha() {
  if (recaptchaWidgetId.value !== null) {
    window.grecaptcha.reset(recaptchaWidgetId.value);
    recaptchaToken.value = '';
  }
}



onMounted(() => {

    const interval = setInterval(() => {

        if(window.grecaptcha && window.grecaptcha.render) {
            renderRecaptcha();
            clearInterval(interval);
        }

    },100);

    onBeforeMount(() => {
        clearInterval(interval);
    })
})






</script>