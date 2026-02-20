<template>
  <section class="pt-5" id="contact">
    <h1 class="text-center my-4">Contact</h1>

    <div class="contact-section">
      <div class="row align-items-center mt-4">
        <!-- Map -->
        <div class="col-md-6 map-container">
          <iframe
            id="gmap_canvas"
            title="Google Map"
            src="https://maps.google.com/maps?q=centro%20escolar%20university%20manila&t=&z=13&ie=UTF8&iwloc=&output=embed"
            frameborder="0"
            scrolling="no"
            marginheight="0"
            marginwidth="0"
          ></iframe>
        </div>

        <!-- Form -->
        <div class="col-md-6">
          <form @submit.prevent="submitForm">
            <div class="mb-3">
              <input
                v-model.trim="name"
                type="text"
                class="form-control contact-form-control"
                placeholder="First Name M.I. Last Name"
                :disabled="isLoading"
                required
              />
            </div>

            <div class="mb-3">
              <input
                v-model.trim="email"
                type="email"
                class="form-control contact-form-control"
                placeholder="Email"
                :disabled="isLoading"
                required
              />
            </div>

            <div class="mb-3">
              <textarea
                v-model.trim="message"
                class="form-control contact-form-control"
                rows="6"
                placeholder="Message"
                :disabled="isLoading"
                required
              ></textarea>
            </div>

            <div class="form-footer d-flex align-items-center justify-content-between flex-wrap gap-3">
              <div class="social-icons">
                <a
                  href="https://www.linkedin.com/in/charles-babbage-8291a6211/"
                  id="linkedin"
                  target="_blank"
                  rel="noopener noreferrer"
                >
                  <i class="fab fa-linkedin"></i>
                </a>

                <a
                  href="https://gitlab.com/cbabbage0991"
                  id="gitlab"
                  target="_blank"
                  rel="noopener noreferrer"
                >
                  <i class="fab fa-gitlab"></i>
                </a>

                <a
                  href="https://github.com/cbabbage0991"
                  id="github"
                  target="_blank"
                  rel="noopener noreferrer"
                >
                  <i class="fab fa-github"></i>
                </a>
              </div>

              <button type="submit" class="submit-btn px-5" :disabled="isLoading">
                {{ isLoading ? "Sending..." : "Submit" }}
              </button>

              <div class="d-flex justify-content-end mt-2">
              <div ref="recaptchaContainer"></div>
              </div>
            </div>
          </form>
        </div>
      </div>
    </div>
  </section>
</template>

<script setup>
import { ref, onMounted } from "vue";
import { Notyf } from "notyf";
import "notyf/notyf.min.css";

const notyf = new Notyf();

/* -------------------- Web3Forms -------------------- */
const WEB3FORMS_ACCESS_KEY = "92c148aa-5e00-45a2-ab1e-f1f6a6e84be0";
const subject = "New message from Portfolio Contact Form";

/* -------------------- Form State -------------------- */
const name = ref("");
const email = ref("");
const message = ref("");
const isLoading = ref(false);

const resetForm = () => {
  name.value = "";
  email.value = "";
  message.value = "";
};

/* -------------------- reCAPTCHA -------------------- */
const SITE_KEY = "6LfjsHEsAAAAANNa0dgWXsdiZX4NotmtM_B0eTMG";

const recaptchaContainer = ref(null);
const recaptchaWidgetID = ref(null);
const recaptchaToken = ref("");

const onRecaptchaSuccess = (token) => {
  recaptchaToken.value = token;
};

const onRecaptchaExpired = () => {
  recaptchaToken.value = "";
};

const renderRecaptcha = () => {
  if (!window.grecaptcha || !recaptchaContainer.value) return;

  // Prevent rendering twice
  if (recaptchaWidgetID.value !== null) return;

  recaptchaWidgetID.value = window.grecaptcha.render(recaptchaContainer.value, {
    sitekey: SITE_KEY,
    size: "normal",
    callback: onRecaptchaSuccess,
    "expired-callback": onRecaptchaExpired,
  });
};

const resetRecaptcha = () => {
  if (!window.grecaptcha) return;

  if (recaptchaWidgetID.value !== null) {
    window.grecaptcha.reset(recaptchaWidgetID.value);
  }
  recaptchaToken.value = "";
};

onMounted(() => {
  // Wait until grecaptcha is available (script loads async)
  const interval = setInterval(() => {
    if (window.grecaptcha?.render) {
      renderRecaptcha();
      clearInterval(interval);
    }
  }, 100);
});

/* -------------------- Submit -------------------- */
const submitForm = async () => {
  if (isLoading.value) return;

  if (!recaptchaToken.value) {
    notyf.error("Please verify that you are not a robot.");
    return;
  }

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
        subject,
        name: name.value,
        email: email.value,
        message: message.value,

        // Web3Forms expects this key for reCAPTCHA
        "g-recaptcha-response": recaptchaToken.value,
      }),
    });

    const result = await response.json();

    if (!response.ok || !result?.success) {
      throw new Error(result?.message || "Web3Forms request failed");
    }

    notyf.success("Message sent!");
    resetForm();
  } catch (err) {
    console.error(err);
    notyf.error("Failed to send message. Please try again.");
  } finally {
    isLoading.value = false;
    resetRecaptcha();
  }
};
</script>