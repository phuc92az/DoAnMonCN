<template>
    <div class="container">
        <div class="row">
            <div class="col-lg-4">
                <div class="card">
                    <div class="card-body">
                        <div class="input-group">
                            <input @keyup.enter="timKiem" type="text" v-model="search.noi_dung" class="form-control border-primary"
                                placeholder="Soát vé" autocomplete="off">
                             <button v-on:click="timKiem()" class="btn btn-primary">Tìm
                                kiếm</button>
                        </div>
                    </div>
                </div>
            </div>

            <div class="col-lg-8">
                <div v-if="status == 0" class="alert alert-danger text-center pb-4" role="alert">
                    <i class="fa-solid fa-circle-exclamation me-2"></i>Không tìm thấy kết quả phù hợp
                </div>
                <div v-if="status == 1" class="alert alert-success" role="alert">
                    <div class="row p-2">
                        <div class="col-lg-4">
                            <img :src="ve.hinh_anh" class="w-100 rounded-4" alt="">
                        </div>
                        <div class="col-lg-8">
                            <h4><i class="fa-solid fa-ticket me-3"></i>Thông tin vé</h4>
                            <div class="mt-2">
                                <div class="d-flex justify-content-between">
                                    <p class="fw-bold">Mã vé: </p>
                                    <p>{{ ve.ma_ve }}</p>
                                </div>
                                <div class="d-flex justify-content-between">
                                    <p class="fw-bold">Tên phim: </p>
                                    <p>{{ ve.ten_phim }}</p>
                                </div>
                                <div class="d-flex justify-content-between">
                                    <p class="fw-bold">Suất chiếu: </p>
                                    <p>{{ ve.thoi_gian_bat_dau }} - {{ ve.thoi_gian_ket_thuc }}</p>
                                </div>
                                <div class="d-flex justify-content-between">
                                    <p class="fw-bold">Phòng chiếu: </p>
                                    <p>{{ ve.ten_phong }}</p>
                                </div>
                                <div class="d-flex justify-content-between">
                                    <p class="fw-bold">Ghế: </p>
                                    <p>{{ ve.ten_ghe }}</p>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import axios from 'axios';

export default {
    data() {
        return {
            search: {
                noi_dung: '',
            },
            status: null,
            ve: {},
            isLoading: false,
        }
    },
    mounted() {

    },
    methods: {
        timKiem() {
            const q = (this.search.noi_dung || '').trim();
            if (!q) {
                this.$toast.error('Vui lòng nhập mã vé');
                return;
            }
            if (this.isLoading) return;
            this.isLoading = true;
            const token = localStorage.getItem("key_admin");
            const headers = token ? { Authorization: 'Bearer ' + token } : {};
            axios.post("http://127.0.0.1:8000/api/admin/ve/soat-ve", { noi_dung: q }, { headers })
                .then((res) => {
                    console.log('soat-ve res:', res); // xem cấu trúc response
                    if (res.data && res.data.status) {
                        this.ve = res.data.data;
                        this.status = 1;
                        // nếu cần giữ mã để quét liên tiếp, bỏ dòng dưới
                        this.search.noi_dung = '';
                    } else {
                        this.status = 0;
                        this.$toast.error(res.data?.message || 'Không tìm thấy vé');
                    }
                })
                .catch((err) => {
                    console.error('soat-ve error:', err);
                    this.status = 0;
                    this.$toast.error(err.response?.data?.message || 'Lỗi kết nối hoặc lỗi server');
                })
                .finally(() => {
                    this.isLoading = false;
                });
        },
    }
}
</script>

<style scoped></style>
