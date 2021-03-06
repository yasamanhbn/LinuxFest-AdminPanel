<template>
    <div>
        <h1>View workshop details with id {{this.$route.params.id}}</h1>
        <div class="col-md-8 order-md-1">
            <form class="needs-validation" @submit.prevent="updateWorkshop()">
                <div class="row">
                    <div class="col-md-6 mb-3">
                        <label for="workshopTitle">Title</label>
                        <input type="text" class="form-control" id="workshopTitle" placeholder="Workshop Title..."
                               v-model="workshop.title">
                    </div>
                    <div class="col-md-6 mb-3">
                        <label for="price">Price(ex : 10000 -> 10 hezar rials)</label>
                        <input type="text" id="price" class="form-control" placeholder="Price..."
                               v-model="workshop.price">
                    </div>
                </div>

                <div class="mb-3">
                    <label for="description">Description:</label>
                    <textarea class="form-control" id="description" placeholder="Description..."
                              v-model="workshop.description"/>
                </div>

                <div class="mb-3">
                    <label for="cap">Capacity (Positive Integer):</label>
                    <input type="number" class="form-control" id="cap" min="0" max="500" placeholder="Capacity..."
                           v-model="workshop.capacity">
                </div>

                <div class="row" v-for="(inputTime, index) in workshop.times" :key="index">
                    <div class="col-md-6">
                        Start Time :
                        <date-picker
                                v-model="inputTime.startTime"
                                :clearable="true"
                                type="datetime"
                                format="YYYY-MM-DD HH:mm"
                                display-format="dddd jDD jMMMM jYYYY HH:mm"
                                input-class="form-control"/>
                    </div>
                    <div class="col-md-6">
                        End Time :
                        <date-picker
                                v-model="inputTime.endTime"
                                :clearable="true"
                                type="datetime"
                                format="YYYY-MM-DD HH:mm"
                                display-format="dddd jDD jMMMM jYYYY HH:mm"
                                input-class="form-control"/>
                    </div>
                </div>
                <hr class="mb-4">

                <label for="teachers">Select Teachers(Required , Can Select Multiple Teachers):</label>
                <select id="teachers" class="custom-select" multiple>
                    <option v-for="teacher in teachers" :key="teachers.indexOf(teacher)" :value="teacher.id">
                        {{teacher.fullName}}
                    </option>
                </select>

                <button class="btn btn-primary btn-lg btn-block" type="submit">Update Current Workshop</button>
            </form>

            <hr>
            <button class="btn btn-info" @click="addTime()">Add time</button>
            <button class="btn btn-warning" @click="removeTime()">Remove time</button>
            <hr>
            <button class="btn btn-secondary" @click="printInput()">View Input</button>

            <h4>List of Participants:</h4>
            <vue-csv-downloader
                class="btn btn-md btn-info mt-3 mb-3"
                :data="participants"
                :fields="exportFieldsAll"
                :download-name="workshop.title + '-FULLDATA-' + (new Date()) + '.csv'"
                >
                Download this list as excel with phone number
            </vue-csv-downloader>
            <br>
            <vue-csv-downloader
                class="btn btn-md btn-info mt-3 mb-3"
                :data="participants"
                :fields="exportFieldsEssentials"
                :download-name="workshop.title + '-ESSENTIALS-' + (new Date()) + '.xls'"
                >
                Download this list as excel essentials only
            </vue-csv-downloader>


            <table class="table table-sm table-hover table-border">
                <tr>
                    <th>#</th>
                    <th>FirstName</th>
                    <th>LastName</th>
                    <th>Email</th>
                    <th>Student Num</th>
                    <th>Actions</th>
                </tr>
                <tr v-for="(participant, index) in participants" :key="index">
                    <th>{{index + 1}}</th>
                    <th>{{participant.firstName }}</th>
                    <th>{{participant.lastName }}</th>
                    <th>{{participant.email}}</th>
                    <th>{{participant.studentNumber}}</th>
                    <th>
                        <button @click="deleteUserFromWorkshop(participant._id)"><i class="material-icons">close</i>
                        </button>
                        <router-link :to="'/users/' + participant._id"><i class="material-icons">person</i>
                        </router-link>
                    </th>
                </tr>
            </table>

            <hr>

            <form @submit.prevent="addSelectedUsersToWorkshop()">
                <label for="users">Select Users to Add(Required , Can Select Multiple Teachers):</label>
                <select id="users" class="custom-select" multiple v-model="selectedUsersToAdd" size=10>
                    <option v-for="user in availableUsersToAdd" :key="availableUsersToAdd.indexOf(user)"
                            :value="user._id">{{user.firstName}} {{user.lastName}} | {{user.email}}
                    </option>
                </select>
                <button class="btn btn-md btn-info mt-3">Add Selected Users to Workshop</button>
            </form>

            <div class="row mt-3 mb-3">
                <div class="col-md-12">
                    <form @submit.prevent="uploadWorkshopAlbum()">
                        <input v-for="(imageField, index) in availableImageFields" :key="index" type="file" class="form-control" ref="file"
                               @change="handleFileUpload()">
                        <button class="btn btn-info">Upload Images</button>
                    </form>
                    <button @click="addImageFieldForUpload()">Add Image Field</button>
                    <button @click="removeImageFieldForUpload()">Remove Image Field</button>
                </div>
            </div>

            <button @click="closeRegistration()" v-if="workshop.isRegOpen">Close Registration</button>
            <button @click="openRegistration()" v-else>Open Registration</button>
        </div>
    </div>
</template>

<script>
    import axios from 'axios'
    import jalali from 'jalali-moment'
    import VuePersianDateTimePicket from "vue-persian-datetime-picker";
    import VueCsvDownloader from 'vue-csv-downloader';

    export default {
        name: "WorkshopProfile",
        components: {
            datePicker: VuePersianDateTimePicket,
            VueCsvDownloader
        },
        data() {
            return {
                workshop: {},
                participants: [],
                currentTeachers: [],
                allTeachers: [],
                selectedTeachers: [],
                users: [],
                selectedUsersToAdd: [],
                availableTeachers: [],
                availableImageFields: [],
                files: [],
                exportFieldsAll : [
                    'firstName',
                    'lastName',
                    'phoneNumber',
                    'studentNumber',
                    'email'
                ],
                exportFieldsEssentials : [
                  'firstName',
                  'lastName',
                  'studentNumber',
                  'email'
                ]

            }
        },
        methods: {

            async closeRegistration() {
                let result = await axios.patch(this.$store.getters.workshopsApi + 'manage/' + this.workshop._id, {"isRegOpen": false}, {
                    headers: {
                        "Authorization": "Bearer " + this.$store.getters.token,
                    }
                });
                console.log(result)
            },

            async openRegistration() {
                let result = await axios.patch(this.$store.getters.workshopsApi + 'manage/' + this.workshop._id, {"isRegOpen": true}, {
                    headers: {
                        "Authorization": "Bearer " + this.$store.getters.token,
                    }
                });
                console.log(result);
            },

            removeImageFieldForUpload() {
                this.availableImageFields.pop();
            },

            addImageFieldForUpload() {
                this.availableImageFields.push(this.availableImageFields.length);
            },

            getJalali(date) {
                return jalali(date);
            },

            printInput() {
                console.log(this.workshop);
            },

            addTime() {
                this.workshop.times.push({
                    startTime: "",
                    endTime: "",
                    _id: this.$route.params.id
                })
            },

            uploadWorkshopAlbum: function () {
                let formData = new FormData();
                formData.append('pictures', this.files);
                axios({
                    url: this.$store.getters.workshopsApi + 'pic/album/' + this.$route.params.id,
                    method: "POST",
                    headers: {
                        'Authorization': 'Bearer ' + this.$store.getters.token,
                        'Content-Type': 'multipart/form-data'
                    },
                    data: formData,
                }).then(response => {
                    console.log(response);
                    console.log('SUCCESS ALBUM UPLOAD')
                }).catch(error => {
                    console.log(error);
                    console.log(error.response);
                })
            },

            handleFileUpload() {
                console.log("handle file upload called");
                console.log(this.$refs.file);
                this.files = [];
                for (let i = 0; i < this.$refs.file.length; i++) {
                    this.files.push(this.$refs.file[i].files[0])
                }
                console.log(this.files);
            },
            removeTime() {
                this.workshop.times.pop();
            },

            updateWorkshop() {
                console.log("update workshop with id ", this.$route.params.id);
                delete this.workshop._id;
                delete this.workshop.__v;
                delete this.workshop.album;
                delete this.workshop.participantsNumber;

                for (let i = 0; i < this.workshop.times.length; i++) {
                    this.workshop.times[i]._id = this.$route.params.id;
                }

                console.log("update payload", this.workshop);
                axios({
                    url: this.$store.getters.workshopsApi + "manage/" + this.$route.params.id,
                    method: "patch",
                    headers: {
                        Authorization: "Bearer " + this.$store.getters.token,
                    },
                    data: this.workshop
                }).then(response => {
                    console.log(response.data);
                    this.getCurrentWorkshop();
                }).catch(error => {
                    console.log(error.response);
                })
            },

            getCurrentWorkshop() {
                axios({
                    url: this.$store.getters.workshopsApi + "manage/" + this.$route.params.id,
                    method: "get",
                    headers: {
                        Authorization: "Bearer " + this.$store.getters.token,
                    }
                }).then(response => {
                    this.workshop = response.data.workshop;
                    this.participants = response.data.participants;
                    this.teachers = response.data.teachers;
                }).catch(error => {
                    console.log(error.response);
                })
            },

            getAllUsers() {
                axios({
                    url: this.$store.getters.usersApi,
                    method: 'GET',
                    headers: {
                        "Authorization": "Bearer " + this.$store.getters.token,
                        "Content-Type": "application/json"
                    }
                }).then(response => {
                    this.users = response.data;
                }).catch(error => {
                    console.log(error);
                    if (error.response)
                        console.log(error.response)
                }).finally(() => {

                })
            },

            deleteUserFromWorkshop(userId) {
                console.log("delete user with id ", userId, " from workshop");
                axios({
                    url: this.$store.getters.workshopsApi + "manage/" + this.$route.params.id + "/user/" + userId,
                    method: 'delete',
                    headers: {
                        "Authorization": "Bearer " + this.$store.getters.token,
                        "Content-Type": "application/json"
                    }
                }).then(response => {
                    console.log(response);
                    this.getCurrentWorkshop();
                }).catch(error => {
                    console.log(error);
                    if (error.response)
                        console.log(error.response)
                }).finally(() => {

                })
            },

            addSelectedUsersToWorkshop() {
                console.log('selected users to add ', this.selectedUsersToAdd);
                let promises = [];
                for (let i = 0; i < this.selectedUsersToAdd.length; i++) {
                    promises.push(axios({
                        url: this.$store.getters.workshopsApi + 'manage/' + this.$route.params.id + '/user/' + this.selectedUsersToAdd[i],
                        method: "put",
                        headers: {
                            Authorization: "Bearer " + this.$store.getters.token
                        }
                    }))
                }

                Promise.all(promises).then((value) => {
                    console.log(value);
                    console.log('all promises done');
                    this.getCurrentWorkshop();
                }).catch(error => {
                    console.log(error);
                    console.log("some error occured");
                })
            },

            isUserIdInParticipants(userIdToSearch) {
                for (let i = 0; i < this.participants.length; i++) {
                    if (this.participants[i]._id == userIdToSearch) {
                        return true;
                    }
                }
                return false;
            },

            getAvailableTeachers() {
                axios({
                    url: this.$store.getters.teachersApi,
                    method: "GET",
                    headers: {
                        "Authorization": "Bearer " + this.$store.getters.token,
                        "Content-Type": "application/json"
                    }
                }).then(response => {
                    console.log(response);
                    this.availableTeachers = response.data;
                    console.log('available teachers ', this.availableTeachers);
                }).catch(error => {
                    console.log(error.response);
                })
            },

            teacherIsInWorkshopTeachers(teacherId) {
                for (let i = 0; i < this.workshop.teachers.length; i++) {
                    if (this.workshop.teachers[i]._id === teacherId)
                        return true
                }
                return false;
            }
        },

        created() {
            this.getCurrentWorkshop();
            this.getAllUsers();
            this.getAvailableTeachers();
        },

        computed: {
            availableUsersToAdd() {
                return this.users.filter(user => {
                    return !this.isUserIdInParticipants(user._id);
                })
            }
        }
    }
</script>

<style scoped>

</style>