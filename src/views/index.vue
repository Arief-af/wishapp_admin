<template>
    <div>
        <ul class="flex space-x-2 rtl:space-x-reverse">
            <li>
                <a href="javascript:;" class="text-primary hover:underline">Dashboard</a>
            </li>
            <li class="before:content-['/'] ltr:before:mr-2 rtl:before:ml-2">
                <span>Pemulihan Akun</span>
            </li>
        </ul>

        <div class="mapouter">
            <div class="gmap_canvas w-full">
                <iframe
                    class="gmap_iframe w-full mt-5 rounded-lg"
                    frameborder="0"
                    scrolling="no"
                    marginheight="0"
                    marginwidth="0"
                    src="https://maps.google.com/maps?width=600&amp;height=400&amp;hl=en&amp;q=insitut nasional bandung&amp;t=&amp;z=18&amp;ie=UTF8&amp;iwloc=B&amp;output=embed"
                ></iframe>
            </div>
            
        </div>
        <div class="pt-5">
            <div class="grid w-full gap-6 mb-6">
                <div class="panel h-full xl:col-span-2">
                    <div class="flex items-center justify-between dark:text-white-light mb-5">
                        <h5 class="font-semibold text-lg">Data Pemulihan Akun</h5>
                    </div>
                    <div class="relative">
                        <apexchart
                            v-if="chartData && chartData.counts.length > 0"
                            height="325"
                            :options="chartOption"
                            :series="[{ name: 'Data', data: chartData.counts }]"
                            class="bg-white dark:bg-black rounded-lg overflow-hidden"
                        >
                        </apexchart>

                        <!-- Loader -->
                        <div v-else class="min-h-[325px] grid place-content-center bg-white-light/30 dark:bg-dark dark:bg-opacity-[0.08]">
                            <span class="animate-spin border-2 border-black dark:border-white !border-l-transparent rounded-full w-5 h-5 inline-flex"></span>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <div class="table-responsive">
            <table>
                <thead>
                    <tr>
                        <th>NIK</th>
                        <th>Email</th>
                        <th>Ibu Kandung</th>
                        <th class="text-center">Send Recovery</th>
                    </tr>
                </thead>
                <tbody>
                    <template v-for="data in users" :key="data.id">
                        <tr>
                            <td class="whitespace-nowrap">{{ data.nik }}</td>
                            <td>{{ data.email }}</td>
                            <td>{{ data.ibu_kandung }}</td>
                            <td class="text-center">
                                <button @click="onSubmit(data.email)">Send</button>
                            </td>
                        </tr>
                    </template>
                </tbody>
            </table>
        </div>
    </div>
</template>

<script setup>
    import axios from 'axios';
    import { ref as vueRef, onMounted, computed, onBeforeMount } from 'vue';
    import { initializeApp } from 'firebase/app';
    import { getDatabase, ref as dbRef, onValue } from 'firebase/database';
    import apexchart from 'vue3-apexcharts';

    // Firebase config and initialization
    const firebaseConfig = {
        apiKey: 'AIzaSyBsG6t85orjj_UePLCFPA1oC2igSI-9OsU',
        authDomain: 'wishapp-8a6dc.firebaseapp.com',
        databaseURL: 'https://wishapp-8a6dc-default-rtdb.firebaseio.com',
        projectId: 'wishapp-8a6dc',
        storageBucket: 'wishapp-8a6dc.appspot.com',
        messagingSenderId: '493613167399',
        appId: '1:493613167399:web:9eb3d4f7325c1533bc2f02',
    };

    const app = initializeApp(firebaseConfig);
    const database = getDatabase(app);

    const users = vueRef([]);

    const fetchRealtimeData = () => {
        const usersRef = dbRef(database, 'users');
        onValue(usersRef, (snapshot) => {
            if (snapshot.exists()) {
                const data = snapshot.val();
                users.value = Object.entries(data).map(([key, value]) => ({
                    id: key,
                    ...value,
                }));
            } else {
                users.value = [];
            }
        });
    };

    const onSubmit = async (emailData) => {
        try {
            const response = await axios.put('/api/user/recovery', {
                email: emailData,
            });

            // Assuming the response structure contains a `data` property with a `message`
            if (response.data && response.data.message) {
                alert(response.data.message);
            } else {
                alert('An unexpected response structure.');
            }
        } catch (error) {
            console.error('Error during recovery request:', error);
            alert('Something went wrong. Please try again later.');
        }
    };

    onMounted(() => {
        fetchRealtimeData();
    });

    const processData = (data) => {
        const monthlyData = {};
        for (const item of data) {
            const createdAt = new Date(item.created_at);
            const month = createdAt.getFullYear() + '-' + (createdAt.getMonth() + 1).toString().padStart(2, '0');
            if (monthlyData[month]) {
                monthlyData[month]++;
            } else {
                monthlyData[month] = 1;
            }
        }
        return {
            months: Object.keys(monthlyData),
            counts: Object.values(monthlyData),
        };
    };

    const chartData = computed(() => processData(users.value));

    import { useAppStore } from '@/stores/index';
    import { email } from '@vuelidate/validators';
    import { useRouter } from 'vue-router';
    const store = useAppStore();
    const router = useRouter();
    onBeforeMount(() => {
        if (!store.isLoggedIn) {
            router.push('/login');
        }
    });
    const chartOption = computed(() => {
        const isDark = store.theme === 'dark' || store.isDarkMode;
        const isRtl = store.rtlClass === 'rtl';
        return {
            chart: {
                height: 325,
                type: 'area',
                fontFamily: 'Nunito, sans-serif',
                zoom: { enabled: false },
                toolbar: { show: false },
            },
            dataLabels: { enabled: false },
            stroke: { show: true, curve: 'smooth', width: 2, lineCap: 'square' },
            dropShadow: { enabled: true, opacity: 0.2, blur: 10, left: -7, top: 22 },
            colors: isDark ? ['#2196f3', '#e7515a'] : ['#1b55e2', '#e7515a'],
            markers: {
                discrete: [
                    { seriesIndex: 0, dataPointIndex: 6, fillColor: '#1b55e2', strokeColor: 'transparent', size: 7 },
                    { seriesIndex: 1, dataPointIndex: 5, fillColor: '#e7515a', strokeColor: 'transparent', size: 7 },
                ],
            },
            labels: chartData.value.months,
            xaxis: {
                axisBorder: { show: false },
                axisTicks: { show: false },
                crosshairs: { show: true },
                labels: { offsetX: isRtl ? 2 : 0, offsetY: 5, style: { fontSize: '12px', cssClass: 'apexcharts-xaxis-title' } },
            },
            yaxis: {
                tickAmount: 7,
                labels: {
                    formatter: (value) => value * 1, // Show value in tens instead of thousands
                    offsetX: isRtl ? -30 : -10,
                    offsetY: 0,
                    style: { fontSize: '12px', cssClass: 'apexcharts-yaxis-title' },
                },
                opposite: isRtl,
            },

            grid: {
                borderColor: isDark ? '#191e3a' : '#e0e6ed',
                strokeDashArray: 5,
                xaxis: { lines: { show: true } },
                yaxis: { lines: { show: false } },
                padding: { top: 0, right: 0, bottom: 0, left: 0 },
            },
            legend: {
                position: 'top',
                horizontalAlign: 'right',
                fontSize: '16px',
                markers: { width: 10, height: 10, offsetX: -2 },
                itemMargin: { horizontal: 10, vertical: 5 },
            },
            tooltip: {
                marker: { show: true },
                x: { show: false },
            },
            fill: {
                type: 'gradient',
                gradient: { opacityFrom: 0.7, opacityTo: 0.1 },
            },
        };
    });
</script>
