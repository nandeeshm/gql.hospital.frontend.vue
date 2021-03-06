<template>
    <div class="patient-dashboard-container">
        <section-header :title="patientName"></section-header>
        <div class="patient-dashboard-wrapper">
            <dashboard-section 
                v-for="sectionName in Object.keys(sections)" 
                :name=sections[sectionName].name
                :icon=sections[sectionName].icon
                :data=sections[sectionName].data
                :key="`patient-dashboard-${sectionName}`" />
        </div>
    </div>
</template>

<script>
import { mapActions } from 'vuex';

import SectionHeader from '../../../common/components/SectionHeader';
import Section from '../components/Section/views/Section.vue';

import * as gql from "../graphql";

export default {
    data: () => ({
        patientId: '',
        patientName: '',
        patientAge: '',
        socialCareNumber: '',
        sections: {
            patientData: {
                name: 'Patient data',
                icon: 'address-card',
                data: []
            },
            reports: {
                name: 'Reports',
                icon: 'file',
                data: []
            },
            treatments: {
                name: 'Treatments',
                icon: 'capsules',
                data: []
            },
            analytics: {
                name: 'Analytics',
                icon: 'tint',
                data: []
            },
            doctors: {
                name: 'Doctors',
                icon: 'stethoscope',
                data: []
            },
            appointments: {
                name: 'Appointments',
                icon: 'calendar',
                data: []
            }
        }
            
    }),
    porps: [
        'patientId'
    ],
    components: {
        sectionHeader: SectionHeader,
        dashboardSection: Section
    },
    methods: {
        ...mapActions([
            'setSelectedPatient'
        ]),
        processPatientData(patientData) {
            this.patientId = patientData.id;
            this.patientName = `${patientData.name} ${patientData.surname}`;
            this.patientAge = (new Date()).getFullYear() - (new Date(patientData.birthDate)).getFullYear();
            this.socialCareNumber = patientData.socialCareNumber;

            this.sections.patientData.data = this._groupPatientData(patientData);
            this.sections.reports.data = patientData.medicalHistory.reports.map(this._parseReportData);

            this.$store.dispatch('setSelectedPatient', patientData);
        },
        _groupPatientData(patientData) {
            return [
                {
                    title: 'Social care number',
                    subtitle: patientData.socialCareNumber
                },
                {
                    title: 'Genre',
                    subtitle: patientData.genre
                },
                {
                    title: 'Birth date',
                    subtitle: `${patientData.birthDate} | ${this.$services.User.calculateAge(patientData.birthDate)} yo.`
                },
            ];
        },
        _parseReportData(report) {
            // TODO: Include a new field which shows the last visit for the doctor who is authenticated.
            //  This field can be named lastComeOn.
            //  It can be calculated serching in the reports, what was the last created for the authenticated doctor for this patient.
            return {
                title: report.reportType,
                subtitle: report.department,
                complement: report.reportDate,
                linkTo: `/report/${report.id}`
            }
        }
    },
    async beforeMount() {
        try {
            let patientData = (await this.$apollo.query(
                gql.getPatientById(this.$route.params.patientId)
            )).data.patient;
            
            if (patientData.__typename !== 'ApiError') {
                this.processPatientData(patientData);
            } else {
                this.$toastr.error(
                    patientData.description,
                    patientData.message
                );
            }
        } catch (error) {
            this.$toastr.error(
                error.message,
                "Getting patient data"
            );
        }
    }
};
</script>

<style lang="scss" scoped>
.patient-dashboard-container {
    position: absolute;
    width: 100%;
    height: 100%;
    top: 0;
    left: 0;
    padding: 0px 20px 25px;
    box-sizing: border-box;
    display: flex;
    flex-direction: column;
    background-color: $main-background;
    .patient-dashboard-wrapper {
        position: relative;
        width: 100%;
        height: 100%;
        padding: 10px 0;
        box-sizing: border-box;
        display: flex;
        flex-direction: row;
        flex-wrap: wrap;
        justify-content: space-between;
        align-content: flex-start;
    }
}
</style>

