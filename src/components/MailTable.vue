<template>
   <table class="mail-table">
    <tbody>
      <tr 
         v-for="email in unarchivedEmails" 
         :key="email.id" 
         :class="['clickable', email.read ? 'read' : '']" 
         @click="readEmail(email)">
        <td>
          <input type="checkbox" />
        </td>
        <td>{{ email.from }}</td>
        <td>
          <p><strong>{{ email.subject }}</strong> - {{ email.body }}</p>
        </td>
        <td class="date">{{ format(new Date(email.sentAt), 'dd MMMM, yyyy') }}</td>
        <td><button @click="archiveEmail(email)">Archive</button></td>
      </tr>
    </tbody>
  </table>
  <div v-if="!!openedEmail">
     <MailView :email="openedEmail" />
  </div>
</template>

<script>
import { format } from 'date-fns'
import { ref } from 'vue'
import { MailView } from '@/components/MailView.vue'
import axios from 'axios'

export default {
  async setup() {
      const { data: emails } = await axios.get('http://localhost:3000/emails') 
    return { 
      format, 
      "emails": ref(emails),
      openedEmail: null
    }
  },
  components: {
     MailView
  },
  computed: {
    sortedEmails() {
      const compareFn = (e1, e2) => e1.sentAt < e2.sentAt ? 1 : -1
      return this.emails.sort(compareFn)
    }, 
    unarchivedEmails() {
      const isNotArchived = e => !e.archived
      return this.sortedEmails.filter(isNotArchived)
    }
  },
  methods: {
     readEmail(email) {
      this.openedEmail = email
      console.log(email)
        if(!email.read) {
           email.read = true
           this.updateEmail(email)
        }
     },
     archiveEmail(email) {
        if(!email.archived) {
           email.archived = true
           this.updateEmail(email)
        }
     },
     updateEmail(email) {
      axios.put(`http://localhost:3000/emails/${email.id}`, email)
     }
  }
}
</script>

<style>

</style>