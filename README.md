# realralphg.github.io
Testing Deployment - EmployeeMgr app with firebase/firestore test database - No Auth



Codes and Guide building for EmployeeMgr Project
.......... by yours truly! ..............

Creating Firebase Account

console.firebase.google.com
make sure you are logged in a google account
Click add a project - enterproject name
Need analytics? - Click!
Click Develop - Select Database
Click Firestore - Create Database
Specify Production or Test mode
Production - Authentication needed
Test - No need for Authentication 
Select Cloud Firestore Location (Cant Change After)

Now start collection - Collection Name or ID
Create documents (rows) and specify all fields within

Go set up your project in VSCode
Then npm install firebase --save


Now go buld project 
in Components ... create 2 JS files
- firebaseConfig.js
- firebaseInit.js

Go to your firebase account
Click - add firebase to your webapp
Register your app
Copy the code insode firebaseConfig Object
Go paste in your code where you created firebaseConfig.js

export default {

//paste here

}

Go to firebaseInit and do this!

import firebase from  'firebase/app'
import 'firebase/firestore'
import firebaseConfig from './firebaseConfig'
const firebaseApp = firebase.initializeApp(firebaseConfig)

export default firebaseApp.firestore()

AND WE ARE DONE CONFIGURING FIREBASE


On Your Project 
Components ... create
- Navbar.vue
- Dashboard.vue
- NewEmployee.vue
- EditEmployee.vue
- ViewEmployee.vue

Now Create Routes
npm install vue-router --save

...THEN Goto ....................main.js
import VueRouter from 'vue-router'


Vue.use(VueRouter);
const router = new VueRouter({
//Register Routes here after creating in separate file - routes.js in SRC
  
})

Go to routes.js

import Dashboard from './components/Dashboard'
import NewEmployee from './components/NewEmployee'
import EditEmployee from './components/EditEmployee'
import ViewEmployee from './components/ViewEmployee'

export default [
    {
        path: '/', 
        name: 'dashboard',
        component: Dashboard
    },
    {
        path: '/new',
        name: 'new-employee',
        component: NewEmployee
    },
    {
        path: '/edit/:employee_id', 
        name:'edit-employee',
        component: EditEmployee
    },
    {
        path: '/:employee_id', 
        name: 'view-employee',
        component: ViewEmployee
    },
]



Return to .........................main.js
import Routes from 'routes'

Nest imported Routes in router instance earlier created
routes: Routes

Then Nest "router" in new Vue instance

Good! Go To App.vue and nest <router-view/>



..................... NOW .................. Let's add some Style

Not directly - I'm already used to Quasar
Let's use CDNs here from:

(1) MaterializeCss.Com - Get Started - CDN

Copy Link to index.html - <head>
 <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/materialize/1.0.0/css/materialize.min.css">

Copy script to index.html - <body>
 <script src="https://cdnjs.cloudflare.com/ajax/libs/materialize/1.0.0/js/materialize.min.js"></script>

(2) jQuery - Materialize JS needs jQuery as a dependency 
- Code.jQuery.com - jQuery 3.x - Uncompressed
- Copy Script to index.html body
<script
src="http://code.jquery.com/jquery-3.5.0.js"
integrity="sha256-r/AaFHrszJtwpe+tHyNi/XCfMxYpbsRg2Uqn0x3s2zc="
crossorigin="anonymous"></script>


(3) FontAwesome CDN
- Search for fontawesome CDN bootstrap
- Font Awesome Css - HTML
- Copy link to index.html and paste in <head>

<link href="https://stackpath.bootstrapcdn.com/font-awesome/4.7.0/css/font-awesome.min.css" rel="stylesheet" integrity="sha384-wvfXpqpZZVQGK6TAh5PVlGOfQNHSoD2xbE+QkPxCAFlNEevoEH3Sl0sibVcOQVnN" crossorigin="anonymous">

......................................

Create Navbar in Navbar.vue
<template>
    <nav class="nav-wrapper purple">
        <div class="container">
            <router-link to="/" class="brand-logo center"> Employee Manager App </router-link>
        </div>
    </nav>
</template>


Import Navbar in App.vue and Register it
import Navbar from './components/Navbar.vue'

<Script>
export default {
components:{

   Navbar

}

Nest it in...

<template>
<div>
  <Navbar/>
  <div class="container">
    <router-view></router-view>
  </div>
</div>
</template>

=========================================================

Add the Floating Add Button to Dashboard

  <div class="fixed-action-btn">
      <router-link to="/new" class="btn-floating button-large blue">
          <i class="fa fa-plus"></i>
      </router-link>
  </div>


On Dashboard Script 

<script>
import db from './firebaseInit'
    export default {
        name: 'dashboard',
        data() {
            return {
                employees: []
            }
        },
        
        created() {
            db.collection('employees').orderBy('employee_id').get().then(querySnapshot => {
                    querySnapshot.forEach(doc => {
                        const data = {
                            'id': doc.id, //bcos it's auto-generated
                            'employee_id': doc.data().employee_id,
                            'name': doc.data().name,
                            'dept': doc.data().dept,
                            'position': doc.data().position,
                        }
                        this.employees.push(data)
                    })
                })
        }
    }
</script>

=========================================================

Let's Now display these data in Dashboard template

 <!-- This list and classes here are from Materialize -->
        <ul class="collection with-header"> 
            <li class="collection-header"><h4>Employees Dashboard</h4></li>
            <li class="collection-item" v-for="employee in employees" :key="employee.id">
                <div class="chip"> {{employee.dept}}</div>
                {{employee.employee_id}} : {{employee.name}}
                
                <!-- Lets add an icon in each list to click <router-link>-->
                <router-link class="secondary-content" :to="{name: 'view-employee', 
                params: {employee_id: employee.employee_id}}">
                    <i class="fa fa-eye"></i>
                </router-link>
            </li>                
        </ul>

=============================================================

TIME TO ViewEmployee - We Added Delete and Back
- beforeRouteEnter(3args){} - collect data in vm
- watch: {} 
- fetchData(){} - collect data in actual variables - this.name
==============================================================

<script>
    import db from './firebaseInit'
    export default {
        name: 'view-employee',
        data() {
            return {
                employee_id: null,
                name: null,
                dept: null,
                position: null
            }
        }, 

        beforeRouteEnter(to, from, next){
            db.collection('employees').where('employee_id', '==', 
            to.params.employee_id).get().then(querySnapshot => {
                querySnapshot.forEach(doc => {
                    next(vm => {
                        vm.employee_id = doc.data().employee_id
                        vm.name = doc.data().name
                        vm.dept = doc.data().dept
                        vm.position = doc.data().position
                    })
                })
            })
        },

        watch: {
            '$route': 'fetchData'
        },

        methods: {
            fetchData () {
                db.collection('employees').where('employee_id', '==',
                 this.$route.params.employee_id).get().then(querySnapshot =>{
                     querySnapshot.forEach(doc => {
                         this.employee_id = doc.data().employee_id
                         this.name = doc.data().name
                         this.dept = doc.data().dept
                         this.position = doc.data().position
                    })
                 })
            },

            deleteEmployee(){
                if(confirm('Really, Delete?')){
                    db.collection('employees').where('employee_id', '==', 
                    this.$route.params.employee_id).get().then(querySnapshot => {
                        querySnapshot.forEach(doc => {
                            doc.ref.delete()
                            this.$router.push('/')
                        })
                    })
                }
            }
        }
    }
</script>

<style scoped>
button{
    margin-left: 10px
}

</style>

===================================================================
- IN <template> ViewEmployee 
- Note: Classes employed are from Materialize
===================================================================

<template>
    <div id='view-employee'>
        <!-- View page for single employees -->
        <h3>View Employee</h3>
        <ul class="collection with-header">
            <li class="collection-header"><h4> {{name}} </h4></li>
            <li class="collection-item">Employee ID#: {{employee_id}}</li>
            <li class="collection-item">Department: {{dept}}</li>
            <li class="collection-item">Position: {{position}}</li>
        </ul>

        <!-- Buttons for Back and Delete -->
        <router-link to="/" class="btn grey"> Back </router-link>
        <button class="btn red" @click="deleteEmployee"> Delete </button>

 	<!-- Floating Button for Edit -->
        <div class="fixed-action-btn">
            <router-link :to="{name: 'edit-employee', params:{employee_id: employee_id}}" 
            class="btn-floating button-large teal">
                <i class="fa fa-pencil"></i>
            </router-link>
        </div>
    </div>
</template>

====================================================================
- Now Add NEW EMPLOYEE

====================================================================
<script>
    import db from './firebaseInit'
    export default {
        data(){
            return{
                employee_id: null,
                name: null,
                dept: null,
                position: null
            }
        }, 

        methods: {
            saveForm(){
                db.collection('employees').add({
                    employee_id: this.employee_id,
                    name: this.name,
                    dept: this.dept,
                    position: this.position
                }).then(() => { 
                    this.$router.push('/')
                })
            }
        }
    
    }
</script>

<style scoped>
button{
    margin-right: 10px
}

</style>

=================================================================
- In <template> New Employee
=================================================================

<template>
    <div id='new-employee'>
        <h3>New Employee</h3>
        <div class="row">
            <form @submit.prevent="saveForm" class="col s12">
                <div class="row">
                    <div class="input-field col s12">
                        <input type="text" v-model="employee_id" required>
                        <label> Employee ID#</label>
                    </div>
                </div>

                <div class="row">
                    <div class="input-field col s12">
                        <input type="text" v-model="name" required>
                        <label> Employee Name</label>
                    </div>
                </div>

                <div class="row">
                    <div class="input-field col s12">
                        <input type="text" v-model="dept" required>
                        <label> Department</label>
                    </div>
                </div>

                <div class="row">
                    <div class="input-field col s12">
                        <input type="text" v-model="position" required>
                        <label> Position</label>
                    </div>
                </div>

                <button class="btn" type="submit"> Save </button>
                <router-link to="/" class="btn red">Cancel</router-link>
            </form>
        </div>
    </div>
</template>

====================================================================
Time to Edit Employee - First Nest a button in ViewEmployee to fire an Edit method
- beforeRouterEnter(){} - watch: {} - fetchData(){} - update(){}
====================================================================

<script>
    import db from './firebaseInit'
    export default {
        data() {
            return {
                employee_id: null,
                name: null,
                dept: null,
                position: null
            }
        }, 

        beforeRouteEnter(to, from, next){
            db.collection('employees').where('employee_id', '==', 
            to.params.employee_id).get().then(querySnapshot => {
                querySnapshot.forEach(doc => {
                    next(vm => {
                        vm.employee_id = doc.data().employee_id
                        vm.name = doc.data().name
                        vm.dept = doc.data().dept
                        vm.position = doc.data().position
                    })
                })
            })
        },

        watch: {
            '$route': 'fetchData'
        },

        methods: {
            fetchData () {
                db.collection('employees').where('employee_id', '==',
                 this.$route.params.employee_id).get().then(querySnapshot =>{
                     querySnapshot.forEach(doc => {
                         this.employee_id = doc.data().employee_id
                         this.name = doc.data().name
                         this.dept = doc.data().dept
                         this.position = doc.data().position
                    })
                 })
            },

            updateForm(){
                db.collection('employees').where('employee_id', '==',
                 this.$route.params.employee_id).get().then(querySnapshot =>{
                     querySnapshot.forEach(doc => {
                        doc.ref.update({
                            employee_id: this.employee_id,
                            name: this.name,
                            dept: this.dept,
                            position: this.position
                        }).then(() => {
                            this.$router.push({name:'view-employee', 
                            params: {employee_id: this.employee_id}})
                        })
                    })
                 })   
            }
        }
     
    }
</script>

<style scoped>
button{
    margin-right: 10px
}

</style>

=================================================================
- In <template> Edit Employees
=================================================================

<template>
    <div id='edit-employee'>
        <h3>Edit Employee</h3>
        <div class="row">
            <form @submit.prevent="updateForm" class="col s12">
                <div class="row">
                    <div class="input-field col s12">
                        <input type="text" v-model="employee_id" required>
                    </div>
                </div>

                <div class="row">
                    <div class="input-field col s12">
                        <input type="text" v-model="name" required>
                    </div>
                </div>

                <div class="row">
                    <div class="input-field col s12">
                        <input type="text" v-model="dept" required>
                    </div>
                </div>

                <div class="row">
                    <div class="input-field col s12">
                        <input type="text" v-model="position" required>
                    </div>
                </div>

                <button class="btn" type="submit"> Update </button>
                <router-link to="/" class="btn red">Cancel</router-link>
            </form>
        </div>
    </div>
</template>

=========================================================================
----- WE'RE ALMOST DONE -----
----- Time for PRODUCTION ---
----- Let's host on GitHub --
=========================================================================
- npm run build

- This creates a dist folder with the: 
- index.html file & all static assets
- the DIST folder is uploaded to our web host
- So the DIST folder will be our "git repository" for our github website

=========================================================================
- Go to your GitHub Account and create a repository
- name repository as username.github.io (realralphg.github.io)
- Add a Description 
- Create Repo
=========================================================================
- Go to your Project Folder, Open DIST, Cut and HIDE content elsewhere temporarily
(Don't Worry --- you'll paste them back afterwards)
- Right click on Dist Folder to open Comment Terminal there "Git Bash here"

- Go to pages.github.com
- Copy code to Clone the Repository and paste in Git Bash terminal: see below

git clone https://github.com/username/username.github.io . (edit to your username)

git clone https://github.com/realralphg/realralphg.github.io . (edited) - enter to run

Note => The " ." at the end is to prevent it from creating another folder in DIST

- Paste back contents removed before into DIST...

==========================================================================
Now back in Git Bash Terminal 
 - git add .    (Notice the dot(.) again
 - git commit -am 'initial commit'
 - git push -u origin master

......... Voilla! ..... We're Done and We are live at realralphg.github.io
==========================================================================
