<template>
  <v-card flat>
    <v-card-title class="d-flex align-center pe-2">
      <v-icon icon="mdi-video-input-component"></v-icon> &nbsp;
      Customer Data
      <v-spacer></v-spacer>

      <v-text-field
        v-model="search"
        density="compact"
        label="Search"
        prepend-inner-icon="mdi-magnify"
        variant="solo-filled"
        flat
        hide-details
        single-line
      ></v-text-field>

      <v-dialog v-model="editDialog" max-width="500px">
        <v-card>
          <v-card-title>
            <span class="text-h5">Editing Project Details</span>
          </v-card-title>

          <v-card-text>
            <v-container>
              <v-row>
                <v-col cols="12" sm="6">
                  <v-autocomplete
                    v-model="editedItem.assignedTO"
                    :items="assignedToList"
                    :min-length="3"
                    label="Assigned To"
                    clearable>
                  </v-autocomplete>
                </v-col>

                <v-col cols="12" sm="6">
                  <v-select
                    v-model="editedItem.contact_channel"
                    :items="contactChannelList"
                    label="Contact Channel">
                  </v-select>
                </v-col>

                <v-col cols="12" sm="6">
                  <v-select
                    v-model="editedItem.status"
                    :items="statusList"
                    label="Status">
                  </v-select>
                </v-col>

                <v-col cols="12" sm="6">
                  <v-select
                    v-model="editedItem.attachments"
                    :items="attachments"
                    label="Attachments">
                  </v-select>
                </v-col>

              </v-row>

              <v-row>
                <v-col cols="12" sm="12">
                  <v-textarea
                    v-model="editedItem.last_comment"
                    :items="last_comment"
                    label="last Comment">
                  </v-textarea>
                </v-col>
              </v-row>

              <v-row>
                <!-- Zaktualizowane pole dla Interaction Date -->
                <v-col cols="12" sm="6">
                  <v-text-field
                    v-model="editedItem.interactionDate"
                    label="Interaction Date"
                    type="date">
                  </v-text-field>
                </v-col>

                <v-col cols="12" sm="6">
                  <v-text-field
                    v-model="editedItem.dueDate"
                    label="Due Date"
                    type="date">
                  </v-text-field>
                </v-col>

                <v-col cols="12" sm="6">
                  <v-text-field
                    v-model="editedItem.interactionTime"
                    label="Interaction Time"
                    type="time">
                  </v-text-field>
                </v-col>

                <v-col cols="12" sm="6">
                  <v-text-field
                    v-model="editedItem.dueTime"
                    label="Due Time"
                    type="time">
                  </v-text-field>
                </v-col>
              </v-row>
            </v-container>
          </v-card-text>

          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="blue darken-1" text @click="closeEdit">Cancel</v-btn>
            <v-btn color="blue darken-1" text @click="saveEdit">Save</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-card-title>

    <v-divider></v-divider>

    <v-row>
      <v-col cols="12" sm="2">
        <v-autocomplete
          v-model="filters.assignedTO"
          :items="assignedToList"
          :min-length="4"
          label="Filter by Assigned To"
          clearable
          multiple
          @change="applyFilters"
        ></v-autocomplete>
      </v-col>
      <v-col cols="12" sm="2">
        <v-autocomplete
          v-model="filters.contact_channel"
          :items="contactChannelList"
          label="Filter by Contract Channel"
          clearable
          multiple
          @change="applyFilters"
        ></v-autocomplete>
      </v-col>
      <v-col cols="12" sm="2">
        <v-autocomplete
          v-model="filters.status"
          :items="statusList"
          label="Filter by Status"
          clearable
          multiple
          @change="applyFilters"
        ></v-autocomplete>
      </v-col>

      <v-col cols="12" sm="6">
        <v-row>
          <v-col cols="6">
            <v-text-field
              v-model="filters.interactionDate.start"
              label="Interaction Date Start"
              type="date"
              clearable
              @change="applyFilters"
            ></v-text-field>
          </v-col>
          <v-col cols="6">
            <v-text-field
              v-model="filters.interactionDate.end"
              label="Interaction Date End"
              type="date"
              clearable
              @change="applyFilters"
            ></v-text-field>
          </v-col>
        </v-row>
      </v-col>
      <!-- Powtórz dla innych kolumn datowych -->
    </v-row>

    <v-progress-linear
      v-if="loading"
      color="blue"
      indeterminate
    ></v-progress-linear>

    <v-data-table
      v-model:search="search"
      :headers="headers"
      :items="items"
      class="elevation-1"
      item-value="name"
      @row-clicked="openModal"
      :loading="loading"
      loading-text="Loading... Please wait."
    >
      <template v-slot:item.status="{ item }">
        <v-chip :color="getStatusColor(item.status)">
          {{ item.status }}
        </v-chip>
      </template>

      <template v-slot:item.actions="{ item }">
        <v-row>
          <v-icon
            small
            class="mr-2"
            @click="editItem(item)"
          >
            mdi-pencil
          </v-icon>
          <v-icon
            small
            class="mr-2"
            @click="showDetails(item)"
          >
            mdi-eye
          </v-icon>
          <v-icon
            class="mr-2"
            small
            @click="deleteItem(item)"
          >
            mdi-delete
          </v-icon>
        </v-row>

      </template>

      <template v-slot:item.last_comment="{ item }">
        <div>
    <span v-if="!item.expandedComment">
      {{ truncateComment(item.last_comment) }}
    </span>
          <span v-else>
      {{ item.last_comment }}
    </span>
          <v-icon small @click.stop="toggleComment(item)">
            {{ item.expandedComment ? 'mdi-chevron-up' : 'mdi-chevron-down' }}
          </v-icon>
        </div>
      </template>

    </v-data-table>

    <v-dialog v-model="modalOpen" persistent max-width="600px">
      <v-card>
        <v-card-title class="text-h5">
          {{ selectedCustomer.first_name + ' ' + selectedCustomer.last_name }}
        </v-card-title>
        <v-card-text>
          <v-container class="pa-0">
            <v-row>
              <v-col cols="12" md="6">
                <div class="py-1"><strong>First Name:</strong> {{ selectedCustomer.first_name }}</div>
                <div class="py-1"><strong>Last Name:</strong> {{ selectedCustomer.last_name }}</div>
                <div class="py-1"><strong>Gender:</strong> {{ selectedCustomer.gender }}</div>
                <div class="py-1"><strong>Country Code:</strong> {{ selectedCustomer.country_code }}</div>
              </v-col>
              <v-col cols="12" md="6">
                <div class="py-1"><strong>Email:</strong> {{ selectedCustomer.email }}</div>
                <div class="py-1"><strong>Phone:</strong> {{ selectedCustomer.phone }}</div>
                <div class="py-1"><strong>Birthday:</strong> {{ (selectedCustomer.birthday) }}</div>
              </v-col>
            </v-row>
          </v-container>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="blue darken-1" text @click="modalOpen = false">Close</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-card>
</template>

<script>
import jsonData from '../data/MOCK_DATA.json'; // Adjust the path as necessary theres test json data with low size

export default {
  data() {
    return {
      search: '',
      assignedToList: [],
      contactChannelList: [],
      statusList: [],
      items: jsonData.map(item => ({
        id: item.id,
        contact_channel: item.contact_channel,
        status: item.status,
        attachments: item.attachments,
        last_comment: item.last_comment,
        interactionDate: this.formatDateTime(item.interaction_creation_date),
        dueDate: this.formatDateTime(item.due_date),
        assignedTO: item.assignedTO,
        customer: {
          first_name: item.customer.first_name,
          last_name: item.customer.last_name,
          email: item.customer.email,
          phone: item.customer.phone,
          gender: item.customer.gender,
          country_code: item.customer.country_code,
          birthday: this.formatDateTime(item.customer.birthday),
        },
      })),
      headers: [
        {align: 'start', key: 'id', title: 'ID', sortable: true},
        {key: 'assignedTO', title: 'Assigned To', sortable: true},
        {key: 'contact_channel', title: 'Contact Channel', sortable: true},
        {key: 'status', title: 'Status', sortable: true},
        {key: 'attachments', title: 'Attachments', sortable: true},
        {key: 'last_comment', title: 'Last Comment', sortable: true},
        {key: 'interactionDate', title: 'Interaction Date', sortable: true},
        {key: 'dueDate', title: 'Due Date', sortable: true},
        {align: 'start', key: 'actions', title: 'Actions', sortable: false},
      ],
      modalOpen: false,
      editDialog: false,
      selectedCustomer: {},
      editedIndex: -1,
      editedItem: {
        id: '',
        contact_channel: '',
        status: '',
        attachments: '',
        last_comment: '',
        interactionDate: '',
        dueDate: '',
        assignedTO: '',
      },
      filters: {
        assignedTO: '',
        contact_channel: '',
        status: '',
        attachments: '',
        last_comment: '',
        interactionDate: '',
        dueDate: '',
      },
    };
  },
  created() {
    this.assignedToList = [...new Set(this.items.map(item => item.assignedTO))];
    this.contactChannelList = [...new Set(this.items.map(item => item.contact_channel))];
    this.statusList = [...new Set(this.items.map(item => item.status))];
    this.attachments = [...new Set(this.items.map(item => item.attachments))];
    this.last_comment = [...new Set(this.items.map(item => item.last_comment))];
  },
  watch: {
    'filters.assignedTO': function (newVal) {
      console.log('Filter assignedTO changed:', newVal);
      this.applyFilters();
    },
    'filters.contact_channel': function (newVal) {
      console.log('Filter contact_channel changed:', newVal);
      this.applyFilters();
    },
    'filters.status': function (newVal) {
      console.log('Filter status changed:', newVal);
      this.applyFilters();
    },
    'filters.interactionDate.start': function (newVal) {
      console.log('Filter interactionDate.start changed:', newVal);
      this.applyFilters();
    },
    'filters.interactionDate.end': function (newVal) {
      console.log('Filter interactionDate.end changed:', newVal);
      this.applyFilters();
    },
  },
  methods: {
    truncateComment(comment) {
      const maxLength = 100; // Adjust as needed
      return comment.length > maxLength ? comment.substr(0, maxLength) + '...' : comment;
    },

    toggleComment(item) {
      item.expandedComment = !item.expandedComment;
    },
    applyFilters() {
      this.items = jsonData.filter(item => {
        console.log("Przed filtracją", item);

        if (this.filters.assignedTO.length && !this.filters.assignedTO.includes(item.assignedTO)) return false;
        if (this.filters.contact_channel.length && !this.filters.contact_channel.includes(item.contact_channel)) return false;
        if (this.filters.status.length && !this.filters.status.includes(item.status)) return false;

        let interactionDate = new Date(item.interaction_creation_date);
        let filterStartDate = this.filters.interactionDate.start ? new Date(this.filters.interactionDate.start) : new Date('1970-01-01');
        let filterEndDate = this.filters.interactionDate.end ? new Date(this.filters.interactionDate.end) : new Date('2999-12-31');
        if (interactionDate < filterStartDate || interactionDate > filterEndDate) return false;

        console.log("Po filtracji", item);
        return true;
      });
    },


    getStatusColor(status) {
      switch (status) {
        case 'waiting':
          return 'amber';
        case 'inProgress':
          return 'blue';
        case 'toTreat':
          return 'orange';
        case 'finished':
          return 'green';
        case 'reserved':
          return 'purple';
        default:
          return 'grey';
      }
    },
    openModal(item) {
      this.selectedCustomer = item.customer;
      this.modalOpen = true;
    },

    editItem(item) {
      this.editedIndex = this.items.indexOf(item);
      this.editedItem = Object.assign({}, item);

      let [interactionDate, interactionTime] = item.interactionDate.split(' ');
      this.editedItem.interactionDate = interactionDate;
      this.editedItem.interactionTime = interactionTime;

      let [dueDate, dueTime] = item.dueDate.split(' ');
      this.editedItem.dueDate = dueDate;
      this.editedItem.dueTime = dueTime;

      this.editDialog = true;
    },

    formatDateTime(date) {
      const options = {
        year: 'numeric', month: 'short', day: 'numeric',
        hour: '2-digit', minute: '2-digit', second: '2-digit',
        timeZoneName: 'short'
      };
      return new Date(date).toLocaleString('en-US', options);
    },


    closeEdit() {
      this.editDialog = false;
      this.$nextTick(() => {
        this.editedItem = {
          id: '',
          contact_channel: '',
          status: '',
          attachments: '',
          last_comment: '',
          interactionDate: '',
          dueDate: '',
          interactionTime: '',
          dueTime: '',
          assignedTO: '',
        };
        this.editedIndex = -1;
      });
    },

    saveEdit() {
      let interactionDateTime = new Date(`${this.editedItem.interactionDate}T${this.editedItem.interactionTime}`);
      this.editedItem.interactionDate = this.formatDateTime(interactionDateTime);

      let dueDateTime = new Date(`${this.editedItem.dueDate}T${this.editedItem.dueTime}`);
      this.editedItem.dueDate = this.formatDateTime(dueDateTime);

      if (this.editedIndex > -1) {
        Object.assign(this.items[this.editedIndex], this.editedItem);
      } else {
        this.items.push(this.editedItem);
      }

      this.closeEdit();
    },

    showDetails(item) {
      this.selectedCustomer = item.customer;
      this.modalOpen = true;
    },
  }
};
</script>
<style scoped>
.v-data-table tr {
  max-height: 100px;
  overflow: hidden;
  display: block;
  text-overflow: ellipsis;
}


.v-data-table th {
  max-width: 200px !important;
}

.v-data-table .v-row {
  flex-wrap: unset;
}

.v-progress-linear {
  color: aliceblue;
}

</style>
