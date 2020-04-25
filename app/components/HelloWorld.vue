<template>
  <Page class="page">
    <ActionBar title="Geolocation" class="action-bar" />
    <ScrollView>
      <StackLayout class="home-panel">
        <Image src="~/images/map-marker-icon.png" height="60" />
        <Button
          :text="active==true?'Stop tracking':'Start tracking'"
          @tap="start"
          class="btn btn-primary"
        />

        <Label :text="'Latitude: ' + latitude" class="lbl" />
        <Label :text="'Longitude: ' + longitude" class="lbl" />
        <Label :text="'Altitude: '+altitude + 'm'" class="lbl" />
        <Label :text="'Speed: ' + speed + 'km/h'" class="lbl" />
      </StackLayout>
    </ScrollView>
  </Page>
</template>

<script>
const geolocation = require("nativescript-geolocation");
const { Accuracy } = require("tns-core-modules/ui/enums");
import axios from "axios/dist/axios";

export default {
  data() {
    return {
      latitude: "",
      longitude: "",
      speed: "",
      altitude: "",
      device: "gsm",
      active: false
    };
  },
  methods: {
    start() {
      this.active = !this.active;
      this.track();
    },
    track() {
      if (this.active) {
        const sec = 3;
        this.getLocation().then(res => {
          this.latitude = res.latitude;
          this.longitude = res.longitude;
          this.speed = res.speed * 3.6;
          this.altitude = res.altitude;
          const payload = {
            latitude: this.latitude,
            longitude: this.longitude,
            speed: this.speed,
            altitude: this.altitude,
            device: this.device
          };
          axios
            .post(
              "https://indy-bap-backend.herokuapp.com/api/locations",
              payload
            )
            .then(() => {
              console.log("succes");
            });
        });
        setTimeout(this.track, sec * 1000);
      } else {
        (this.latitude = ""),
          (this.longitude = ""),
          (this.speed = ""),
          (this.altitude = "");
      }
    },
    getLocation() {
      return geolocation.getCurrentLocation({
        desiredAccuracy: Accuracy.High,
        timeout: 3000,
        updateTime: 2000,
        minimumUpdateTime: 3000
      });
    }
  },
  mounted() {
    geolocation.enableLocationRequest();
  }
};
</script>

<style scoped>
.home-panel {
  vertical-align: center;
  font-size: 20;
  margin: 15;
}

.description-label {
  margin-bottom: 15;
}
</style>