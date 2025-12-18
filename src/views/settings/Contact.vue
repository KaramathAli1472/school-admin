<template>
  <div class="contact-page">
    <!-- Header -->
    <div class="contact-header">
      <h1>Contact Us</h1>
      <p>
        Have questions, concerns, or need support?  
        Please reach out to us and we will respond as soon as possible.
      </p>
    </div>

    <div class="contact-wrapper">
      <!-- Contact Info -->
      <div class="contact-info">
        <h3>School / App Support</h3>

        <p>
          📧 <strong>Email:</strong><br />
          support@schoolapp.com
        </p>

        <p>
          📞 <strong>Phone:</strong><br />
          +91 XXXXX XXXXX
        </p>

        <p>
          🕒 <strong>Support Hours:</strong><br />
          Monday – Saturday (9:00 AM – 5:00 PM)
        </p>

        <p class="note">
          For privacy-related requests, data deletion, or export,
          please clearly mention it in the message.
        </p>
      </div>

      <!-- Contact Form -->
      <div class="contact-form-card">
        <h3>Send Us a Message</h3>

        <form @submit.prevent="submitForm">
          <div class="form-group">
            <label>Full Name</label>
            <input type="text" v-model="form.name" required />
          </div>

          <div class="form-group">
            <label>Email Address</label>
            <input type="email" v-model="form.email" required />
          </div>

          <div class="form-group">
            <label>Subject</label>
            <input type="text" v-model="form.subject" required />
          </div>

          <div class="form-group">
            <label>Message</label>
            <textarea
              rows="4"
              v-model="form.message"
              required
            ></textarea>
          </div>

          <button type="submit">Submit</button>

          <p v-if="success" class="success-msg">
            ✅ Thank you! Your message has been sent successfully.
          </p>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "ContactUs",
  data() {
    return {
      form: {
        name: "",
        email: "",
        subject: "",
        message: ""
      },
      success: false
    }
  },
  methods: {
    submitForm() {
      // 🔒 Future: Save to Firebase / Send Email
      console.log("Contact message:", this.form)

      this.success = true
      this.form = {
        name: "",
        email: "",
        subject: "",
        message: ""
      }

      setTimeout(() => {
        this.success = false
      }, 4000)
    }
  }
}
</script>

<style scoped>
.contact-page {
  background: #f4f6f8;
  min-height: 100vh;
  padding: 32px 16px;
}

.contact-header {
  max-width: 900px;
  margin: 0 auto 32px;
  text-align: center;
}

.contact-header h1 {
  font-size: 2rem;
  color: #1f2937;
}

.contact-header p {
  color: #6b7280;
}

.contact-wrapper {
  max-width: 900px;
  margin: 0 auto;
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
}

.contact-info,
.contact-form-card {
  background: #ffffff;
  border-radius: 12px;
  padding: 20px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.06);
}

.contact-info h3,
.contact-form-card h3 {
  color: #1976d2;
  margin-bottom: 10px;
}

.contact-info p {
  margin-bottom: 10px;
  color: #374151;
}

.note {
  font-size: 0.9rem;
  color: #6b7280;
}

.form-group {
  display: flex;
  flex-direction: column;
  margin-bottom: 12px;
}

label {
  font-weight: 500;
  margin-bottom: 4px;
}

input,
textarea {
  padding: 8px 10px;
  border-radius: 6px;
  border: 1px solid #d1d5db;
}

input:focus,
textarea:focus {
  outline: none;
  border-color: #1976d2;
}

button {
  background: #1976d2;
  color: white;
  border: none;
  padding: 10px;
  border-radius: 6px;
  cursor: pointer;
  width: 100%;
}

.success-msg {
  margin-top: 10px;
  color: #2e7d32;
  font-weight: 500;
  text-align: center;
}

/* Mobile */
@media (max-width: 700px) {
  .contact-wrapper {
    grid-template-columns: 1fr;
  }
}
</style>

