<template>
  <view class="container">
    <view class="header">
      <text class="title">咨询医生</text>
    </view>
    <scroll-view class="doctor-list" scroll-y="true">
      <view v-for="(doctor) in doctors" :key="doctor.doctorId" class="doctor-card">
        <image :src="doctor.avatarUrl || '/static/doctor-avatars/default.jpg'" mode="aspectFill" class="doctor-avatar"
          @tap="viewDoctorProfile(doctor)"></image>
        <view class="doctor-info">
          <text class="doctor-name">{{ doctor.name }}</text>
          <text class="doctor-title">{{ doctor.position || '医生' }}</text>
          <br>
          <text class="doctor-hospital">{{ doctor.workplace || '未知医院' }}</text>
        </view>
        <button :class="['apply-button', getButtonClass(doctor.situation)]" @tap="handleButtonClick(doctor)">
          {{ getButtonText(doctor.situation) }}
        </button>
      </view>
    </scroll-view>
  </view>
</template>

<script>
import { mapState, mapActions } from 'vuex';
import { bindDoctorPatientRelation } from '@/api/relation.js';

export default {
  computed: {
    ...mapState(['doctors']) // 使用 Vuex 的 state
  },
  methods: {
    ...mapActions(['loadDoctors']), // 使用 Vuex 的 actions

    getButtonClass(situation) {
      switch (situation) {
        case 'rejected': return 'rejected-button';
        case 'approved': return 'approved-button';
        case 'pending': return 'pending-button';
        default: return 'apply-button';
      }
    },
    getButtonText(situation) {
      switch (situation) {
        case 'rejected': return '已被拒绝';
        case 'approved': return '我的医生';
        case 'pending': return '申请中';
        default: return '申请成为患者';
      }
    },
    handleButtonClick(doctor) {
      if (doctor.situation === 'approved') {
        uni.navigateTo({
          url: `/pages/doctorChat/doctorChat?doctor=${encodeURIComponent(JSON.stringify(doctor))}`
        });
      } else if (doctor.situation === 'rejected' || doctor.situation === 'pending') {
        uni.showToast({
          title: '您已经申请过该医生',
          icon: 'none'
        });
      } else {
        this.applyDoctor(doctor.doctorId);
      }
    },
    async fetchDoctors() {
      try {
        //if (!this.doctors.length) {
          await this.loadDoctors(); // 加载医生数据
        //}
        //else
        //{
        //  console.log('Doctors already loaded');
        //}

        
      } catch (error) {
        uni.showToast({
          title: '加载医生信息失败',
          icon: 'none',
        });
      }
    },

    viewDoctorProfile(doctor) {
      uni.navigateTo({
        url: `/pages/doctorProfile/doctorProfile?doctor=${encodeURIComponent(JSON.stringify(doctor))}`,
      });
    },

    async applyDoctor(doctorId) {
      try {
        await bindDoctorPatientRelation(doctorId);
        // Update the doctor's situation in the local state
        const doctorIndex = this.doctors.findIndex(d => d.doctorId === doctorId);
        if (doctorIndex !== -1) {
          this.$set(this.doctors[doctorIndex], 'situation', 'pending');
        }
        uni.showModal({
          title: '申请成功',
          content: '您的申请已提交，请在申请记录中查看申请是否通过。',
          showCancel: false
        });
      } catch (error) {
        uni.showToast({
          title: '申请失败，请重试',
          icon: 'none'
        });
      }
    },
  },

  async mounted() {
    await this.fetchDoctors(); // 在组件加载时获取医生数据
  }
};
</script>
<style>
.container {
  display: flex;
  flex-direction: column;
  height: 100vh;
  background-color: #f5f5f5;
}

.header {
  padding: 20px;
  background-color: #ffffff;
  border-bottom: 1px solid #e0e0e0;
}

.title {
  font-size: 24px;
  font-weight: bold;
  color: #333;
}

.doctor-list {
  flex: 1;
  padding: 10px;
}

.doctor-card {
  display: flex;
  align-items: center;
  background-color: #ffffff;
  border-radius: 10px;
  padding: 15px;
  margin-bottom: 10px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.doctor-avatar {
  width: 80px;
  height: 80px;
  border-radius: 40px;
  margin-right: 15px;
}

.doctor-info {
  flex: 1;
}

.doctor-name {
  font-size: 18px;
  font-weight: bold;
  color: #333;
}

.doctor-title {
  font-size: 14px;
  color: #666;
  margin-top: 5px;
}

.doctor-hospital {
  font-size: 14px;
  color: #999;
  margin-top: 5px;
}

.apply-button {
  background-color: #007aff;
  color: #ffffff;
  padding: 8px 12px;
  border-radius: 5px;
  font-size: 14px;
}

.doctor-name {
  margin-right: 8px;
}

.rejected-button {
  background-color: #ff3b30;
  color: #ffffff;
}

.approved-button {
  background-color: #4cd964;
  color: #ffffff;
}

.pending-button {
  background-color: #ffcc00;
  color: #ffffff;
}

.apply-button,
.rejected-button,
.approved-button,
.pending-button {
  padding: 8px 12px;
  border-radius: 5px;
  font-size: 14px;
}
</style>
