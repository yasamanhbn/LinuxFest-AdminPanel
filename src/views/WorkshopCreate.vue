<template>
    <div>
        <h1>Create New Workshop :</h1>
        <div class="col-md-8 order-md-1">
            <form class="needs-validation" @submit.prevent="createNewWorkshop()">
                <div class="row">
                    <div class="col-md-6 mb-3">
                        <label for="workshopTitle">Title</label>   
                        <input type="text" class="form-control" id="workshopTitle" placeholder="Workshop Title..." v-model="inputWorkshop.title">
                    </div>
                    <div class="col-md-6 mb-3">
                        <label for="price">Price(ex : 10000 -> 10 hezar toman)</label>   
                        <input type="text" id="price" class="form-control" placeholder="Price..." v-model="inputWorkshop.price">
                    </div>
                </div>

                <div class="mb-3">
                        <label for="description">Description:</label>   
                        <textarea class="form-control" id="description" placeholder="Description..." v-model="inputWorkshop.description">

                        </textarea>
                </div>

               <div class="mb-3">
                        <label for="cap">Capacity (Positive Integer):</label>   
                        <input type="number" class="form-control" id="cap" min="0" max="500" placeholder="Capacity..." v-model="inputWorkshop.capacity">

                </div>
                
                <label for="teachers">Select Teachers(Required , Can Select Multiple Teachers):</label>
                <select id="teachers" class="custom-select" multiple v-model="inputWorkshop.teachers">
                    <option v-for="teacher in availableTeachers" :key="availableTeachers.indexOf(teacher)" :value="teacher._id">{{teacher.fullName}}</option>
                </select>

                <div class="row" v-for="inputTime in inputWorkshop.times">
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
                <button class="btn btn-primary btn-lg btn-block">Create New Workshop</button>
            </form>

            <hr>
            <button class="btn btn-info" @click="addTime()">Add time</button>
            <button class="btn btn-warning" @click="removeTime()">Remove time</button>
            <hr>
            <button class="btn btn-secondary" @click="printInput()">View Input</button>
        </div>
    </div>
</template>

<script>
    import axios from 'axios';
    import jalali from 'jalali-moment';
    import VuePersianDateTimePicket from "vue-persian-datetime-picker";
export default {
    name : "WorkshopCreate",
    components : {
        datePicker : VuePersianDateTimePicket,
    },
    data() {
        return {
            inputWorkshop : {
                title : "Ghamsar OS",
                capacity : "20",
                price : "100000",
                description: "Ghamsar os is leading unix based systems.",
                teachers : [],
                startTime : "",
                endTime : "",
                times : []
            },
            availableTeachers : [],
        }
    },
    methods : {
        getJalali(date) {
            return jalali(date);
        },

        printInput() {
            console.log(this.inputWorkshop);
        },

        addTime() {
            this.inputWorkshop.times.push({
                startTime : "",
                endTime : "",
            })
        },

        removeTime() {
            this.inputWorkshop.times.pop();
        },

        backendizeTeachersStructure() {
            let teachers = [];
            for(let i = 0; i < this.inputWorkshop.teachers.length ; i++) {
                teachers.push({"id" : this.inputWorkshop.teachers[i]})
            }
            return teachers;
        },

        createNewWorkshop: function() {
            //create new workshop
            console.log("Create new workshop");
            this.inputWorkshop.teachers = this.backendizeTeachersStructure(this.inputWorkshop.teachers);
            console.log(this.inputWorkshop); 

            axios({
                url : this.$store.getters.workshopsApi,
                method : "POST",
                data :  this.inputWorkshop,
                headers : {
                    Authorization : "Bearer " + this.$store.getters.token,
                    "Content-Type" : "application/json"
                }
            }).then(response => {
                console.log(response);
            }).catch(error => {
                console.log(error.response);
            })
        },
        getAvailableTeachers() {
            axios({
                url : this.$store.getters.teachersApi,
                method : "GET",
                headers : {
                    "Authorization" : "Bearer " + this.$store.getters.token,
                    "Content-Type" : "application/json"
                }
            }).then(response => {
                console.log(response);
                // this.availableTeachers = response.data;
                for(let i = 0 ; i < response.data.length ; i++) {
                    this.availableTeachers.push(response.data[i].teacher);
                }
                console.log(this.availableTeachers);
            }).catch(error => {
                console.log(error.response);
            })
        }
    },
    created() {
        this.getAvailableTeachers();
    }
}
</script>

<style scoped>

</style>