// v-model has a two-way binding while v-bind can only communicate one-way.

<template>
  <div class="tile is-ancestor">
    <div class="tile is-vertical is-9">
      <div class="tile is-parent">
        <article class="tile is-child">
          <div class="content">
            <div class="form-button">
              <button class="button is-primary" @click="this.formModeToggle">
                {{ formModeButton }}
              </button>
            </div>
            <fieldset v-bind="{
              disabled: this.formMode == 'disabled',
            }">
              <div class="field is-horizontal">
                <div class="field-label is-normal">
                  <label class="label">Email</label>
                </div>
                <div class="field-body">
                  <div class="field">
                    <div class="control">
                      <input class="input" type="email" v-bind:value="email"  disabled />
                    </div>
                  </div>
                </div>
              </div>

              <div class="field is-horizontal">
                <div class="field-label is-normal">
                  <label class="label">Mobile Number</label>
                </div>
                <div class="field-body">
                  <div class="field">
                    <div class="control">
                      <input class="input"  v-model="mobile" placeholder="Mobile Number" @input="enforcePhoneFormat()" required/>
                    </div>
                  </div>
                </div>
              </div>

              <div class="field is-horizontal">
                <div class="field-label is-normal">
                  <label class="label">Role</label>
                </div>
                <div class="field-body">
                  <div class="field">
                    <div class="control">
                      <input class="input" type="text" v-bind:value="role" placeholder="Role" disabled />
                    </div>
                  </div>
                </div>
              </div>

              <!-- <div class="field is-horizontal">
                <div class="field-label is-normal">
                  <label class="label">Image</label>
                </div>
                <div class="field-body">
                  <div class="file is-medium">
                    <label class="file-label">
                      <input class="file-input" type="file" name="resume" @change="imageFileSelect" />
                      <span class="file-cta">
                        <span class="file-icon">
                          <i class="fas fa-upload"></i>
                        </span>
                        <span class="file-label"> Upload Profile Picture </span>
                      </span>
                    </label>
                  </div>
                </div>
              </div> -->

              <div class="field is-horizontal">
                <div class="field-label is-normal">
                  <label class="label">House Name</label>
                </div>
                <div class="field-body">
                  <div class="field">
                    <div class="control">
                      <input class="input" type="text" v-model="houseName" placeholder="House Name" required/>
                    </div>
                  </div>
                </div>
              </div>

              <div class="field is-horizontal">
                <div class="field-label is-normal">
                  <label class="label">Street Name</label>
                </div>
                <div class="field-body">
                  <div class="field">
                    <div class="control">
                      <input class="input" type="text" v-model="streetName" placeholder="Street Name" required/>
                    </div>
                  </div>
                </div>
              </div>

              <div class="field is-horizontal">
                <div class="field-label is-normal">
                  <label class="label">City</label>
                </div>
                <div class="field-body">
                  <div class="field">
                    <div class="control">
                      <input class="input" type="text" v-model="cityName" placeholder="City" required/>
                    </div>
                  </div>
                </div>
              </div>

              <div class="field is-horizontal">
                <div class="field-label is-normal">
                  <label class="label">PinCode</label>
                </div>
                <div class="field-body">
                  <div class="field">
                    <div class="control">
                      <input class="input" type="number" v-model="pincode" placeholder="Pincode" required />
                    </div>
                  </div>
                </div>
              </div>

            <!-- <div class="field is-horizontal">
                <div class="field-label is-normal">
                  <label class="label">Account Status</label>
                </div>
                <div class="field-body">
                  <div class="field">
                    <div class="control">
                      <input
                        class="input"
                        type="text"
                        v-bind:value="enableStatus"
                        placeholder="Account Status"
                        disabled
                      />
                    </div>
                  </div>
                  </div>
                </div> -->
            </fieldset>
          </div>
        </article>
      </div>
    </div>

    <div class="tile is-parent">
      <article class="tile is-child">
        <div class="profile-content-div">
          <div class="profile-pic-name">
            <div>
              <img class="profile-pic" v-bind:src="imageUrl" />
            </div>

            <div>
              <h1 class="title is-2">{{ fullName }}</h1>
            </div>
          </div>
        </div>
      </article>
    </div>
  </div>
</template>

<style>
.form-button {
  text-align: center;
  margin-bottom: 20px;
}

.form-button button {
  font-size: larger;
  padding-left: 20px;
  padding-right: 20px;
}

.profile-pic-name {
  width: fit-content;
  text-align: center;
}

.profile-pic {
  width: 256px;
  height: 256px;
  border-radius: 50%;
}

.profile-content-div {
  display: flex;
  justify-content: right;
}
</style>

<script>
import axios from "axios";

export default {
  data() {
    return {
      formMode: "disabled",
      formModeButton: "Edit",
      fullName: "",
      email: "",
      mobile: "",
      role: "",
      houseName: "",
      streetName: "",
      cityName: "",
      pincode: "",
      enableStatus: "",
      imageUrl: "",

      image: "",
      userId: ''
    };
  },

  mounted() {
    const token = localStorage.getItem("token")
    var base64Url = token.split(".")[1];
    var base64 = base64Url.replace(/-/g, "+").replace(/_/g, "/");
    var jsonPayload = decodeURIComponent(
      window
        .atob(base64)
        .split("")
        .map(function (c) {
          return "%" + ("00" + c.charCodeAt(0).toString(16)).slice(-2);
        })
        .join("")
    );
    let decoded = JSON.parse(jsonPayload);
    let decodedId = decoded.userId
    this.userId = decodedId
    this.getProfile();
  },

  methods: {
    formModeToggle() {
      if (this.formMode == "disabled") {
        this.formMode = "enabled";
        this.formModeButton = "Save";
      } else {
        this.formMode = "disabled";
        this.formModeButton = "Edit";
        this.setProfile();
      }
    },
    enforcePhoneFormat() {
    let x = this.phone_number
      .replace(/\D/g, "")
      .match(/(\d{0,3})(\d{0,3})(\d{0,4})/);

    this.phone_number = !x[2]
      ? x[1]
      : "(" + x[1] + ") " + x[2] + (x[3] ? "-" + x[3] : "");
  },
    imageFileSelect(event) {
      this.$store.commit('setIsLoading', true)
      console.log(event)
      var imageFile = event.target.files[0];
      this.createBase64Image(imageFile)
    },

    createBase64Image(fileObject) {
      var reader = new FileReader();
      reader.onload = (e) => {
        var imageFileData = e.target.result;
        this.image = imageFileData.slice(imageFileData.indexOf(",") + 1);
        console.log(this.image)
        this.uploadImage();
      };
      reader.readAsDataURL(fileObject);
    },

    async uploadImage() {
      var formData = new FormData();
      formData.append("image", this.image)

      const client = axios.create({
        transformRequest: [(data, headers) => {
          // add required "Content-Type" whenever body is defined
          delete headers.common.Authorization
          return data
        }],
      })

      await client
        .post("https://api.imgbb.com/1/upload?key=0f6650dbe5d582897945e5dd899204bd", formData)
        .then((response) => {
          console.log(response)
          var imageData = response.data.data;
          this.imageUrl = imageData.url;
        })
      this.$store.commit('setIsLoading', false)
    },

    async getProfile() {
      axios
        .get("/api/users/shop/getdetails/"+this.userId)
        .then((response) => { 
          var profileData = response.data; 
          // this.fullName = profileData.fullName;
          this.email = profileData.email;
          this.mobile = profileData.phone;
          this.role = profileData.role;
          this.houseName = profileData.house_name;
          this.streetName = profileData.street;
          this.cityName = profileData.city;
          this.pincode = profileData.pincode;
          this.enableStatus = profileData.enableStatus;
          this.imageUrl = profileData.imageUrl;
        })
        .catch((error) => {
          console.log(error);
        });
    },

    async setProfile() {
      var profileData = {
        email: this.email,
        phone: this.mobile,
        profile: this.imageUrl,
        house_name: this.houseName,
        street: this.streetName,
        city: this.cityName,
        pincode: this.pincode
      };

      axios
        .put("/api/users/shop/update/"+this.userId, profileData)
        .then((response) => {
          var profileData = response.data;
          this.fullName = profileData.fullName;
          this.email = profileData.email;
          this.phone = profileData.mobile;
          this.role = profileData.role;
          this.houseName = profileData.house_name;
          this.streetName = profileData.street;
          this.cityName = profileData.city;
          this.pincode = profileData.pincode;
          this.enableStatus = profileData.enableStatus;
          this.imageUrl = profileData.profile;
        })
        .catch((error) => {
          console.log(error);
        });
    },

  },
};
</script>
