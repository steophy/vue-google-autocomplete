<template>
    <span class="input-wrapper" style="height: 40px;">
        <input
            ref="autocomplete"
            type="text"
            :class="classname"
            :id="id"
            :placeholder="placeholder"
            v-model="autocompleteText"
            @focus="onFocus()"
            @blur="onBlur()"
            @change="onChange"
            @keypress="onKeyPress"
        />
        <div
            v-if="autocompleteText"
            class="x-close"
            @click="clear">x</div>
        <div class="lens-img"></div>
    </span>
</template>

<script>
    export default {
        name: 'VueGoogleAutocomplete',

        props: {
          id: {
            type: String,
            required: true
          },

          classname: String,

          placeholder: {
            type: String,
            default: 'Start typing'
          },

          types: {
            type: String,
            default: 'address'
          },

          country: {
            type: [String, Array],
            default: null
          },

          enableGeolocation: {
            type: Boolean,
            default: false
          }
        },

        data: function () {
            return {
                /**
                 * The Autocomplete object.
                 *
                 * @type {Autocomplete}
                 * @link https://developers.google.com/maps/documentation/javascript/reference#Autocomplete
                 */
                autocomplete: null,

                /**
                 * Autocomplete input text
                 * @type {String}
                 */
                autocompleteText: '',
            }
        },

        watch: {
            autocompleteText: function (newVal, oldVal) {
                this.$emit('inputChange', { newVal, oldVal });
            }
        },

        mounted: function() {
          const options = {};

          if (this.types) {
            options.types = [this.types];
          }

          if (this.country) {
            options.componentRestrictions = {
              country: this.country
            };
          }

          this.autocomplete = new google.maps.places.Autocomplete(
                document.getElementById(this.id),
                options
            );

          this.autocomplete.addListener('place_changed', () => {

                let place = this.autocomplete.getPlace();

                if (!place.geometry) {
                  // User entered the name of a Place that was not suggested and
                  // pressed the Enter key, or the Place Details request failed.
                  this.$emit('no-results-found', place);
                  return;
                }

                let addressComponents = {
                    street_number: 'short_name',
                    route: 'long_name',
                    locality: 'long_name',
                    administrative_area_level_1: 'short_name',
                    country: 'long_name',
                    postal_code: 'short_name'
                };

                let returnData = {};

                if (place.address_components !== undefined) {
                    // Get each component of the address from the place details
                    for (let i = 0; i < place.address_components.length; i++) {
                      let addressType = place.address_components[i].types[0];

                      if (addressComponents[addressType]) {
                        let val = place.address_components[i][addressComponents[addressType]];
                            returnData[addressType] = val;
                      }
                    }

                    returnData['latitude'] = place.geometry.location.lat();
                    returnData['longitude'] = place.geometry.location.lng();

                    // return returnData object and PlaceResult object
                    this.$emit('placechanged', returnData, place, this.id);

                    // update autocompleteText then emit change event
                    this.autocompleteText = document.getElementById(this.id).value
                    this.onChange()
                }
           });
        },

        methods: {
            /**
             * When the input gets focus
             */
            onFocus() {
              this.geolocate();
              this.$emit('focus');
            },

            /**
             * When the input loses focus
             */
            onBlur() {
              this.$emit('blur');
            },

            /**
             * When the input got changed
             */
            onChange() {
              this.$emit('change', this.autocompleteText);
            },

            /**
             * When a key gets pressed
             * @param  {Event} event A keypress event
             */
            onKeyPress(event) {
              this.$emit('keypress', event);
            },

            /**
             * Clear the input
             */
            clear() {
              this.autocompleteText = ''
            },

            /**
             * Focus the input
             */
            focus() {
              this.$refs.autocomplete.focus()
            },

            /**
             * Blur the input
             */
            blur() {
              this.$refs.autocomplete.blur()
            },

            /**
             * Update the value of the input
             * @param  {String} value
             */
            update (value) {
              this.autocompleteText = value
            },

            // Bias the autocomplete object to the user's geographical location,
            // as supplied by the browser's 'navigator.geolocation' object.
            geolocate() {
                if (this.enableGeolocation) {
                    if (navigator.geolocation) {
                      navigator.geolocation.getCurrentPosition(position => {
                        let geolocation = {
                          lat: position.coords.latitude,
                          lng: position.coords.longitude
                        };
                        let circle = new google.maps.Circle({
                          center: geolocation,
                          radius: position.coords.accuracy
                        });
                        this.autocomplete.setBounds(circle.getBounds());
                      });
                    }
                }
            }
        }
    }
</script>
<style scoped>
.x-close {
    position: absolute;
    right: 0px;
    font-size: 25px;
    font-family: Lato, sans-serif;
    cursor: pointer;
    font-weight: 300;
    top: 7px;
    border-style: solid;
    border-width: 0 1px 0 0;
    padding-left: 2px;
    height: 26px;
    line-height: 0.8;
    padding-right: 6px;
}

.input-wrapper {
    position: absolute;
    top: -50px;
    right: 30px;
    width: 265px;
    display: flex;
    align-items: center;
    border-style: solid;
    border-width: 0 0 1px 0;
}

.input-wrapper > input {
    border: none;
    width: 245px;
}

.lens-img {
    background-image: url(data:image/png;base64,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);
    background-repeat: no-repeat;
    background-size: 20px;
    width: 19px;
    height: 19px;
    left: 270px;
    position: absolute;
}
</style>
