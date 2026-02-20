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
            </div>
          </form>
        </div>
      </div>
    </div>
  </section>
</template>

<script setup>
import { ref } from "vue";
import { Notyf } from "notyf";
import "notyf/notyf.min.css";

const notyf = new Notyf();

const WEB3FORMS_ACCESS_KEY = "92c148aa-5e00-45a2-ab1e-f1f6a6e84be0";
const subject = "New message from Portfolio Contact Form";

const name = ref("");
const email = ref("");
const message = ref("");
const isLoading = ref(false);

const resetForm = () => {
  name.value = "";
  email.value = "";
  message.value = "";
};

const submitForm = async () => {
  if (isLoading.value) return;

  // simple validation
  if (!name.value || !email.value || !message.value) {
    notyf.error("Please fill in all fields.");
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
      }),
    });

    const result = await response.json();

    if (!response.ok || !result.success) {
      throw new Error(result?.message || "Web3Forms request failed");
    }

    notyf.success("Message sent!");
    resetForm();
  } catch (err) {
    console.error(err);
    notyf.error("Failed to send message. Please try again.");
  } finally {
    isLoading.value = false;
  }
};
</script>