<template>
  <div class="numbering">{{ place_index+1 }}/{{ places.length }}</div>
  <h1>Where is {{ current_place.name }}?</h1>
  <svg
    class="map"
    viewBox="0 0 500 391.78327951109065"
  >
    <defs>
      <linearGradient
        id="gradient"
        :x1="guess_point.x"
        :y1="guess_point.y"
        :x2="place_point.x"
        :y2="place_point.y"
        gradientUnits="userSpaceOnUse"
      >
        <stop offset="0" stop-color="red"/>
        <stop offset="1" stop-color="green"/>
      </linearGradient>
    </defs>
    <use
      href="/map.svg#path7330"
      @click="on_map_click"
    />
    <line
      pathLength="1"
      :class="{ 'revealed': !is_guessing }"
      :x1="guess_point.x"
      :y1="guess_point.y"
      :x2="place_point.x"
      :y2="place_point.y"
      stroke-width="10"
      stroke="url('#gradient')"
      :stroke-linecap="is_guessing ? undefined : 'round'"
    ></line>
    <circle
      :class="{ 'revealed': !is_guessing }"
      :cx="place_point.x"
      :cy="place_point.y"
      r="15"
      fill="green"
    ></circle>
    <circle
      :class="{ 'revealed': !is_guessing }"
      :cx="guess_point.x"
      :cy="guess_point.y"
      r="15"
      fill="red"
    ></circle>
  </svg>
  <div :class="{ 'bottom': true, 'revealed': !is_guessing }">
    <div>{{ feedback }}</div>
    <div>Distance: {{ distance }} km</div>
    <button type="button" @click="next">Next</button>
  </div>
</template>
<script>
export default {
  data: () => ({
    is_guessing: true,
    places: [
      { name: 'Vilnius', lat: 54.6872, lng: 25.2797 },
      { name: 'Kaunas', lat: 54.8985, lng: 23.9036 },
      { name: 'Klaipėda', lat: 55.7106, lng: 21.1356 },
      { name: 'Šiauliai', lat: 55.9333, lng: 23.3167 },
      { name: 'Panevėžys', lat: 55.7294, lng: 24.3569 },
      { name: 'Alytus', lat: 54.3978, lng: 24.0414 },
      { name: 'Marijampolė', lat: 54.5594, lng: 23.3528 },
      { name: 'Utena', lat: 55.5047, lng: 25.5942 },
      { name: 'Tauragė', lat: 55.2525, lng: 22.2892 },
      { name: 'Telšiai', lat: 55.9819, lng: 22.2564 }
    ],
    feedbacks_available: [
      { distance: 10, text: "Great job! You nailed it!" },
      { distance: 50, text: "Nice work! You were pretty close!" },
      { distance: 100, text: "Not bad! You're getting there!" },
      { distance: 150, text: "You missed it by a bit. Keep trying!" },
      { distance: 200, text: "You missed it by quite a bit. Try again!" },
      { distance: Infinity, text: "You missed it by a long shot. Keep practicing!" }
    ],
    place_index: 0,
    guess_point: { x: 20, y: 20 },
  }),
  computed: {
    feedback() {
      for (const f of this.feedbacks_available) {
        if (this.distance < f.distance) return f.text
      }
    },
    distance() {
      return Math.round(
        this.calculate_distance(
          this.current_place,
          this.guess
        )
      )
    },
    current_place() {
      return this.places[this.place_index]
    },
    place_point() {
      return this.gps2xy(this.current_place)
    },
    guess() {
      return this.xy2gps(this.guess_point)
    }
  },
  methods: {
    next() {
      this.is_guessing = true
      this.place_index = (this.place_index + 1) % this.places.length
    },
    on_map_click(e) {
      if (!this.is_guessing) return
      const rect = e.target.parentElement.getBoundingClientRect()
      const scale = 500 / rect.width // svg viewbox width
      this.guess_point = {
        x: scale * (e.clientX - rect.left),
        y: scale * (e.clientY - rect.top),
      }
      this.is_guessing = false
    },
    gps2xy({ lat, lng }) {
      const f = (lat) => {
        const radians = Math.PI / 180;
        return 180 / Math.PI * Math.log(
          Math.tan(Math.PI / 4 + lat * radians / 2)
        );
      };
      const x = lng * 76.31483451625877 + -1573.8412560729162;
      const y = f(lat) * -76.31483451625877 + 5268.019093053688;
      return { x, y }
    },
    xy2gps({ x, y }) {
      const inverse_f = (y) => {
        const radians = Math.PI / 180;
        const lat = 2 * (Math.atan(Math.exp(y / (180 / Math.PI * -1))) - Math.PI / 4) / radians;
        return lat;
      }
      const lat = inverse_f((y - 5268.019093053688) / 76.31483451625877)
      const lng = (x + 1573.8412560729162) / 76.31483451625877;
      return { lat, lng };
    },
    calculate_distance(coord1, coord2) {
      const deg2rad = (deg) => deg * (Math.PI/180)
      const lat1 = coord1.lat
      const lat2 = coord2.lat
      const lon1 = coord1.lng
      const lon2 = coord2.lng
      const R = 6371; // Radius of the earth in km
      const dLat = deg2rad(lat2-lat1);  // deg2rad below
      const dLon = deg2rad(lon2-lon1); 
      const a = Math.sin(dLat/2) * Math.sin(dLat/2) +
        Math.cos(deg2rad(lat1)) * Math.cos(deg2rad(lat2)) * 
        Math.sin(dLon/2) * Math.sin(dLon/2)
        ; 
      const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a)); 
      const d = R * c; // Distance in km
      return d;
    }
  }
}
</script>
<style scoped>
  svg * {
    transform-box: fill-box;
  }
  circle {
    transform: scale(0);
  }
  circle[fill="green"].revealed {
    transition-delay: .7s;
  }
  circle.revealed {
    transform: scale(1);
    transform-origin: center center;
    transition: transform .3s cubic-bezier(0, 3, 1, 1);
  }
  line {
    stroke-dasharray: 1;
    stroke-dashoffset: 1;
  }
  line.revealed {
    stroke-dashoffset: 0;
    transition: stroke-dashoffset .7s ease-out;
  }
  .bottom {
    visibility: hidden;
    opacity: 0;
  }
  .bottom.revealed {
    visibility: visible;
    opacity: 1;
    transition: opacity .2s .7s;
  }
  .numbering {
    font-size: 1.5rem;
  }
  h1 {
    font-size: 2rem;
    margin-top: 0;
  }
  .map {
    width: 300px;
    margin-bottom: 1rem;
  }
  button {
    margin-top: 1rem;
  }
</style>
