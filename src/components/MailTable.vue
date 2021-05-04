<template>
  <table class="mail-table">
    <tbody>
      <tr
        v-for="email in unarchivedEmails"
        :key="email.id"
        :class="['clickable', email.read ? 'read' : '']"
        @click="openEmail(email)"
      >
        <td>
          <input type="checkbox" />
        </td>
        <td>{{ email.from }}</td>
        <td>
          <p>
            <strong>{{ email.subject }}</strong> - {{ email.body }}
          </p>
        </td>
        <td class="date">
          {{ format(new Date(email.sentAt), "dd MMMM, yyyy") }}
        </td>
        <td><button @click="archiveEmail(email)">Archive</button></td>
      </tr>
    </tbody>
  </table>

  <ModalView v-if="openedEmail" @closeModal="openedEmail = null">
    <MailView :email="openedEmail" @changeEmail="changeEmail" />
  </ModalView>
</template>

<script>
import { format } from "date-fns";
import { ref } from "vue";

import MailView from "@/components/MailView.vue";
import ModalView from "@/components/ModalView.vue";
import axios from "axios";

export default {
  async setup() {
    const { data: emails } = await axios.get("http://localhost:3000/emails");
    return {
      format,
      emails: ref(emails),
      openedEmail: ref(null),
    };
  },
  components: {
    MailView,
    ModalView,
  },
  computed: {
    sortedEmails() {
      const compareFn = (e1, e2) => (e1.sentAt < e2.sentAt ? 1 : -1);
      return this.emails.sort(compareFn);
    },
    unarchivedEmails() {
      const isNotArchived = (e) => !e.archived;
      return this.sortedEmails.filter(isNotArchived);
    },
  },
  methods: {
    openEmail(email) {
      this.openedEmail = email;
      if (email) {
        email.read = true;
        this.updateEmail(email);
      }
    },
    archiveEmail(email) {
      if (!email.archived) {
        email.archived = true;
        this.updateEmail(email);
      }
    },
    changeEmail({ toggleRead, toggleArchive, save, closeModal, changeIndex }) {
      const email = this.openedEmail;
      if (toggleRead) email.read = !email.read;
      if (toggleArchive) email.archived = !email.archived;
      if (save) this.updateEmail(email);
      if (closeModal) this.openedEmail = null;
      if(changeIndex) {
         const emails = this.unarchivedEmails
         const currentIndex = emails.indexOf(this.openedEmail)
         const newEmail = emails[currentIndex + changeIndex]
         this.openedEmail(newEmail)
      }
    },
    updateEmail(email) {
      axios.put(`http://localhost:3000/emails/${email.id}`, email);
    },
  },
};
</script>

<style>
</style>