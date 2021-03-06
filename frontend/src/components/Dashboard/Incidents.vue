<template>
  <div class="col-12">
    <div
      v-for="inc in incidents"
      :key="inc.id"
      class="card contain-card mb-4"
    >
      <div class="card-header">
        Incident: {{ inc.title }}
        <button
          class="btn btn-sm btn-danger float-right"
          @click="deleteIncident(inc)"
        >
          <font-awesome-icon icon="times" />
        </button>
      </div>

      <FormIncidentUpdates :incident="inc" />

      <span class="font-2 p-2 pl-3">
        Created: {{ niceDate(inc.created_at) }} | Last Update: {{ niceDate(inc.updated_at) }}
      </span>
    </div>


    <div class="card contain-card">
      <div class="card-header">
        Create Incident
      </div>
      <div class="card-body">
        <form @submit.prevent="createIncident">
          <div class="form-group row">
            <label class="col-sm-4 col-form-label">
              Title
            </label>
            <div class="col-sm-8">
              <input
                id="title"
                v-model="incident.title"
                type="text"
                name="title"
                class="form-control"
                placeholder="Incident Title"
                required
              >
            </div>
          </div>

          <div class="form-group row">
            <label class="col-sm-4 col-form-label">
              Description
            </label>
            <div class="col-sm-8">
              <textarea
                id="description"
                v-model="incident.description"
                rows="5"
                name="description"
                class="form-control"
                required
              />
            </div>
          </div>

          <div class="form-group row">
            <div class="col-sm-12">
              <button
                :disabled="!incident.title || !incident.description"
                type="submit"
                class="btn btn-block btn-primary"
                @click.prevent="createIncident"
              >
                Create Incident
              </button>
            </div>
          </div>
          <div
            id="alerter"
            class="alert alert-danger d-none"
            role="alert"
          />
        </form>
      </div>
    </div>
  </div>
</template>

<script>
import Api from '../../API';

const FormIncidentUpdates = () => import(/* webpackChunkName: "dashboard" */ '@/forms/IncidentUpdates');

export default {
    name: 'Incidents',
    components: { FormIncidentUpdates },
    data () {
        return {
            serviceID: 0,
            incidents: [],
            incident: {
                title: '',
                description: '',
                service: 0,
            }
        };
    },

    created () {
        this.serviceID = Number(this.$route.params.id);
        this.incident.service = Number(this.$route.params.id);
    },

    async mounted () {
        await this.loadIncidents();
    },

    methods: {

        async delete (i) {
            this.res = await Api.incident_delete(i);
            if (this.res.status === 'success') {
                this.incidents = this.incidents.filter(obj => obj.id !== i.id);
                //await this.loadIncidents()
            }
        },
        async deleteIncident (incident) {
            const modal = {
                visible: true,
                title: 'Delete Incident',
                body: `Are you sure you want to delete Incident ${incident.title}?`,
                btnColor: 'btn-danger',
                btnText: 'Delete Incident',
                func: () => this.delete(incident),
            };
            this.$store.commit('setModal', modal);
        },

        async createIncident () {
            this.res = await Api.incident_create(this.serviceID, this.incident);
            if (this.res.status === 'success') {
                this.incidents.push(this.res.output); // this is better in terms of not having to querry the db to get a fresh copy of all updates
                //await this.loadIncidents()
            } // TODO: further error checking here... maybe alert user it failed with modal or so

            // reset the form data
            this.incident = {
                title: '',
                description: '',
                service: this.serviceID,
            };
        },

        async loadIncidents () {
            this.incidents = await Api.incidents_service(this.serviceID);
        }

    }
};
</script>
