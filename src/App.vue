<template>
  <v-app>

    <v-data-table
        :headers="headers"
        :items="workers"
        :search="search"
        sort-by="workers"
        class="elevation-1"
    >
      <template v-slot:top>
        <v-toolbar
            flat
        >
          <v-toolbar-title>List of employees</v-toolbar-title>
          <v-divider
              class="mx-4"
              inset
              vertical
          ></v-divider>
          <v-text-field
              v-model="search"
              append-icon="mdi-magnify"
              label="Filter"
              single-line
              hide-details
          ></v-text-field>
          <v-spacer></v-spacer>
          <v-dialog
              v-model="dialog"
              max-width="500px"
          >
            <template v-slot:activator="{ on, attrs }">
              <v-btn
                  color="primary"
                  dark
                  class="mb-2"
                  v-bind="attrs"
                  v-on="on"
              >
                Add an employee
              </v-btn>
            </template>
            <v-card>
              <v-card-title>
                <span class="text-h5">{{ formTitle }}</span>
              </v-card-title>

              <v-card-text>
                <v-container>
                  <v-row>
                    <v-col
                        cols="12"
                        sm="6"
                        md="4"
                    >
                      <v-text-field

                          v-model="editedItem.firstName"
                          :error-messages="nameErrors"
                          :counter="10"
                          label="firstName"
                          required
                          @input="$v.editedItem.firstName.$touch()"
                          @blur="$v.editedItem.firstName.$touch()"
                      ></v-text-field>
                    </v-col>
                    <v-col
                        cols="12"
                        sm="6"
                        md="4"
                    >
                      <v-text-field
                          v-model="editedItem.lastName"
                          :error-messages="lastNameErrors"
                          :counter="10"
                          label="lastName"
                          required
                          @input="$v.editedItem.lastName.$touch()"
                          @blur="$v.editedItem.lastName.$touch()"
                      ></v-text-field>
                    </v-col>
                    <v-col
                        cols="12"
                        sm="6"
                        md="4"
                    >
                      <v-text-field
                          v-model="editedItem.position"
                          :error-messages="positionErrors"
                          :counter="25"
                          label="Position"
                          required
                          @input="$v.editedItem.position.$touch()"
                          @blur="$v.editedItem.position.$touch()"
                      ></v-text-field>
                    </v-col>
                    <v-col
                        cols="12"
                        sm="6"
                        md="4"
                    >
                      <v-text-field
                          v-model="editedItem.age"
                          :error-messages="ageErrors"
                          label="Age"
                          required
                          @input="$v.editedItem.age.$touch()"
                          @blur="$v.editedItem.age.$touch()"
                      ></v-text-field>
                    </v-col>
                    <v-col
                        cols="12"
                        sm="6"
                        md="4"
                    >
                      <v-checkbox
                          v-model="editedItem.employmentRecord"
                          label="Employment record"
                      ></v-checkbox>
                    </v-col>
                    <v-col
                        cols="12"
                        sm="6"
                        md="4"
                    >
                      <v-text-field
                          v-model="editedItem.salary"
                          :error-messages="salaryErrors"
                          label="Salary"
                          required
                          @input="$v.editedItem.salary.$touch()"
                          @blur="$v.editedItem.salary.$touch()"
                      ></v-text-field>
                    </v-col>
                    <v-col
                        cols="12"
                        sm="6"
                        md="4"
                    >
                      <v-text-field
                          v-model="editedItem.dateOfEmployment"
                          :error-messages="dateErrors"
                          label="Date of employment"
                          required
                          @input="$v.editedItem.dateOfEmployment.$touch()"
                          @blur="$v.editedItem.dateOfEmployment.$touch()"
                      ></v-text-field>
                    </v-col>
                    <v-col
                        cols="12"
                        sm="6"
                        md="4"
                    >
                      <v-text-field
                          v-model="editedItem.rate"
                          :error-messages="rateErrors"
                          label="Rate"
                          required
                          @input="$v.editedItem.rate.$touch()"
                          @blur="$v.editedItem.rate.$touch()"
                      ></v-text-field>
                    </v-col>
                  </v-row>
                </v-container>
              </v-card-text>

              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn
                    color="blue darken-1"
                    text
                    @click="close"
                >
                  Cancel
                </v-btn>
                <v-btn
                    color="blue darken-1"
                    text
                    @click="save"
                >
                  Save
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
          <v-dialog v-model="dialogDelete" max-width="500px">
            <v-card>
              <v-card-title class="text-h5">Are you sure you want to delete this item?</v-card-title>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="closeDelete">Cancel</v-btn>
                <v-btn color="blue darken-1" text @click="deleteItemConfirm">OK</v-btn>
                <v-spacer></v-spacer>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-toolbar>
      </template>
      <template v-slot:[`item.actions`]="{ item }">
        <v-icon
            small
            class="mr-2"
            @click="editItem(item)"
        >
          mdi-pencil
        </v-icon>
        <v-icon
            small
            @click="deleteItem(item)"
        >
          mdi-delete
        </v-icon>
      </template>
      <template v-slot:no-data>
        <v-btn
            color="primary"
            @click="initialize"
        >
          Reset
        </v-btn>
      </template>
    </v-data-table>
    <router-link to="@/components/Inform-page.vue"></router-link>
  </v-app>
</template>

<script>
import { validationMixin } from 'vuelidate'
import { required, maxLength, numeric, alpha } from 'vuelidate/lib/validators'
import moment from 'moment'

export default {
  mixins: [validationMixin],

  validations: {
    editedItem: {
      firstName: { required,
        validRuEng: v => /^[а-яё]*$/i.test(v) || alpha,
        maxLength: maxLength(10) },
      lastName: { required, validRuEng: v => /^[а-яё]*$/i.test(v) || alpha, maxLength: maxLength(10) },
      position: { required, validRuEng: v => /^[а-яё]*$/i.test(v) || alpha, maxLength: maxLength(25) },
      age: { required, numeric, maxLength: maxLength(2) },
      salary: { required, numeric, maxLength: maxLength(6) },
      dateOfEmployment: {
        required,
        validDate: val => moment(val, 'DD.MM.YYYY', true).isValid(),
      },
      rate: { required, validRate: v => v === 'full' || v === 'not full' }
    },
  },



  data: () => ({
    firstName: '',
    lastName: "",
    position: "",
    age: "",
    salary: "",
    dateOfEmployment: "",
    rate: "",

    dialog: false,
    dialogDelete: false,
    search: '',
    headers: [
      {
        text: 'firstName',
        align: 'start',
        sortable: false,
        value: 'firstName',
      },
      { text: 'LastName', sortable: false, value: 'lastName' },
      { text: 'position', value: 'position' },
      { text: 'age', value: 'age' },
      { text: '', value: 'actions', sortable: false },
    ],
    workers: [],
    base: [
      {
        id: 0,
        firstName: "Romin",
        lastName: "Irani",
        position: "Developer",
        age: "27",
        employmentRecord: true,
        salary: "140000",
        dateOfEmployment: "21.01.2020",
        rate: "full"
      },
      {
        id: 1,
        firstName: "Neil",
        lastName: "Irani",
        position: "Developer",
        age: "35",
        employmentRecord: true,
        salary: "150000",
        dateOfEmployment: "15.07.2017",
        rate: "full"
      },
      {
        id: 2,
        firstName: "Tom",
        lastName: "Hanks",
        position: "Program Directory",
        age: "51",
        employmentRecord: true,
        salary: "270000",
        dateOfEmployment: "10.09.2014",
        rate: "full"
      },
      {
        id: 3,
        firstName: "John",
        lastName: "Kelly",
        position: "Network Administrator",
        age: "22",
        employmentRecord: false,
        salary: "25000",
        dateOfEmployment: "15.03.2022",
        rate: "not full"
      },
      {
        id: 4,
        firstName: "Gloria",
        lastName: "Thomas",
        position: "System(s) Enginee",
        age: "43",
        employmentRecord: true,
        salary: "110000",
        dateOfEmployment: "16.11.2021",
        rate: "full"
      },
      {
        id: 5,
        firstName: "Kimberly",
        lastName: " Little",
        position: "Software Developer",
        age: "57",
        employmentRecord: true,
        salary: "340000",
        dateOfEmployment: "12.01.2011",
        rate: "full"
      },
      {
        id: 6,
        firstName: "Elizabeth",
        lastName: "Smith",
        position: "Traine",
        age: "19",
        employmentRecord: false,
        salary: "35000",
        dateOfEmployment: "22.06.2022",
        rate: "not full"
      },
      {
        id: 7,
        firstName: "Steve",
        lastName: "Bates",
        position: "Software Developer",
        age: "38",
        employmentRecord: true,
        salary: "160000",
        dateOfEmployment: "11.03.2018",
        rate: "full"
      },
      {
        id: 8,
        firstName: "Darlene",
        lastName: "Olson",
        position: "Team Leader",
        age: "45",
        employmentRecord: true,
        salary: "350000",
        dateOfEmployment: "15.10.2016",
        rate: "full"
      },
      {
        id: 9,
        firstName: "Pamela",
        lastName: "Hall",
        position: "Tester",
        age: "29",
        employmentRecord: true,
        salary: "50000",
        dateOfEmployment: "12.07.2019",
        rate: "full"
      }
    ],
    editedIndex: -1,
    editedItem: {
      id: Date.now(),
      firstName: "",
      lastName: "",
      position: "",
      age: "",
      employmentRecord: true,
      salary: "",
      dateOfEmployment: "",
      rate: ""
    },
    defaultItem: {
      id: Date.now(),
      firstName: "",
      lastName: "",
      position: "",
      age: "",
      employmentRecord: true,
      salary: "",
      dateOfEmployment: "",
      rate: ""
    },
  }),

  computed: {
    formTitle () {
      return this.editedIndex === -1 ? 'New Item' : 'Edit Item'
    },
    nameErrors () {
      const errors = []
      if (!this.$v.editedItem.firstName.$dirty) return errors
      !this.$v.editedItem.firstName.maxLength && errors.push('Name must be at most 10 characters long')
      !this.$v.editedItem.firstName.validRuEng && errors.push('Accepts only alphabet characters')
      !this.$v.editedItem.firstName.required && errors.push('Name is required')
      return errors
    },
    lastNameErrors () {
      const errors = []
      if (!this.$v.editedItem.lastName.$dirty) return errors
      !this.$v.editedItem.lastName.maxLength && errors.push('Last name must be at most 10 characters long')
      !this.$v.editedItem.lastName.validRuEng && errors.push('Accepts only alphabet characters')
      !this.$v.editedItem.lastName.required && errors.push('Last name is required')
      return errors
    },
    positionErrors () {
      const errors = []
      if (!this.$v.editedItem.position.$dirty) return errors
      !this.$v.editedItem.position.maxLength && errors.push('Position must be at most 25 characters long')
      !this.$v.editedItem.position.validRuEng && errors.push('Accepts only alphabet characters')
      !this.$v.editedItem.position.required && errors.push('Position is required.')
      return errors
    },
    ageErrors () {
      const errors = []
      if (!this.$v.editedItem.age.$dirty) return errors
      !this.$v.editedItem.age.maxLength && errors.push('Age must be at most 2 characters long')
      !this.$v.editedItem.age.numeric && errors.push('Accepts only numerics')
      !this.$v.editedItem.age.required && errors.push('Age is required.')
      return errors
    },
    salaryErrors () {
      const errors = []
      if (!this.$v.editedItem.salary.$dirty) return errors
      !this.$v.editedItem.salary.maxLength && errors.push('Salary must be at most 6 characters long')
      !this.$v.editedItem.salary.numeric && errors.push('Accepts only numerics')
      !this.$v.editedItem.salary.required && errors.push('Salary is required.')
      return errors
    },
    dateErrors () {
      const errors = []
      if (!this.$v.editedItem.dateOfEmployment.$dirty) return errors
      !this.$v.editedItem.dateOfEmployment.required && errors.push('Date is required.')
      !this.$v.editedItem.dateOfEmployment.validDate && errors.push('the date must be in the format "DD.MM.YYYY"')
      return errors
    },
    rateErrors () {
      const errors = []
      if (!this.$v.editedItem.rate.$dirty) return errors
      !this.$v.editedItem.rate.required && errors.push('Rate is required.')
      !this.$v.editedItem.rate.validRate && errors.push('rate must be "full" or "not full"')
      return errors
    },
  },

  watch: {
    dialog (val) {
      val || this.close()
    },
    dialogDelete (val) {
      val || this.closeDelete()
    },
  },

  created () {
    let indexID = 0
    if (localStorage.length) {
      for (let i = 0, length = localStorage.length + 1; i < length; i++) {
        const oldItem = JSON.parse(localStorage.getItem(`${i}`))
        if (!(oldItem === null)) {
          oldItem.id = indexID
          indexID += 1
          this.workers.push(oldItem)
          localStorage.setItem(oldItem.id, JSON.stringify(oldItem))
        }
      }
    } else {
      for (const item of this.base) {
        this.workers.push(item)
        localStorage.setItem(item.id, JSON.stringify(item))
      }
    }
    this.initialize()
  },

  methods: {
    initialize () {
      this.workers
    },

    editItem (item) {
      this.editedIndex = this.workers.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialog = true
    },

    deleteItem (item) {
      this.editedIndex = this.workers.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialogDelete = true
    },

    deleteItemConfirm () {
      let indexID = 0
      this.workers.splice(this.editedIndex, 1)
      localStorage.removeItem(this.editedItem.id)
      for (let i = 0, length = localStorage.length + 1; i < length; i++) {
        const oldItem = JSON.parse(localStorage.getItem(`${i}`))
        localStorage.removeItem(`${i}`)
        this.workers.splice(this.editedIndex, 1)
        if (!(oldItem === null)) {
          oldItem.id = indexID
          indexID += 1
          this.workers.push(oldItem)
          localStorage.setItem(oldItem.id, JSON.stringify(oldItem))
        }
      }
      this.closeDelete()
    },

    close () {
      this.dialog = false
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })
    },

    closeDelete () {
      this.dialogDelete = false
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })
    },

    save () {
      this.$v.$touch()
      if (this.editedIndex > -1 && !this.$v.$invalid) {
        Object.assign(this.workers[this.editedIndex], this.editedItem)
        localStorage.setItem(`${this.editedItem.id}`, JSON.stringify(this.editedItem));
        this.close()
      } else if (!this.$v.$invalid) {
        let item = localStorage.length
        this.editedItem.id = item
        this.workers.push(this.editedItem)
        localStorage.setItem(item, JSON.stringify(this.editedItem))
        this.close()
      }
    },
  },
}
</script>
