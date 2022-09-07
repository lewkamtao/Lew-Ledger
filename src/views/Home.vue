<script lang="ts" setup>
    import { ref, onMounted, computed } from 'vue';
    import { getMonthDate, getHeadDate } from '@/utils/date';
    import moment from 'moment';
    import { ChevronLeft16Filled, ChevronRight16Filled } from '@vicons/fluent';
    import axios from '@/axios/http';

    let dateData = ref(getMonthDate());
    // 获取当天日期对象
    let today = new Date();
    // 获取当前年份
    let curYear = ref(today.getFullYear());
    // 获取当前月份
    let curMonth = ref(today.getMonth() + 1);
    let curDay = ref(today.getDate());

    // 获取当前年份
    let tYear = ref(today.getFullYear());
    // 获取当前月份
    let tMonth = ref(today.getMonth() + 1);
    let tDay = ref(today.getDate());

    let _curDate = Number(moment(`${curYear.value}-${curMonth.value}-${curDay.value}`).format('X'));
    let isloading = ref(false);
    let visible = ref(false);

    let form = ref({
        birthday: '',
        type: '1',
        amount: '',
        purpose: ''
    });

    let group = ref({
        expenditure: '',
        income: '',
        total: ''
    });

    let result = ref([] as any);

    const getClass = computed(() => (item: any) => {
        let dateStr = `${item.year}-${item.month}-${item.showDate}`;
        let _date = Number(moment(dateStr).format('X'));
        if (item.showDate != item.date) {
            return 'not-range-month-item';
        }
        if (_curDate >= _date) {
            if (item?.expand?.total < 0) {
                return 'pass-item cur-month-item expenditure-item';
            }
            if (item?.expand?.total >= 0) {
                return 'pass-item cur-month-item income-item';
            }
            return 'pass-item cur-month-item';
        }
        if (_curDate == _date) {
            return 'today-item pass-item';
        }
        if (item.date == item.showDate) {
            return 'cur-month-item';
        }
    });

    const formatData = () => {
        for (const date_item of dateData.value) {
            let date_item_str = `${date_item.year}-${date_item.month}-${date_item.showDate}`;
            let date_item_num = Number(moment(date_item_str).format('X'));
            if (result.value && result.value.order_group) {
                for (const order_item of result.value.order_group) {
                    order_item.date = moment(order_item.date).format('yyyy-MM-DD');
                    if (Number(moment(order_item.date).format('X')) == date_item_num) {
                        date_item['expand'] = order_item;
                    }
                }
            }
            if (date_item_num <= _curDate) {
                date_item.status = '待填写';
            }
        }
    };

    const next = () => {
        if (isloading.value) {
            return;
        }
        if (curMonth.value == 12) {
            curMonth.value = 1;
            curYear.value += 1;
        } else {
            curMonth.value += 1;
        }
        dateData.value = getMonthDate(curYear.value, curMonth.value);
        getData();
    };

    const prve = () => {
        if (isloading.value) {
            return;
        }
        if (curMonth.value == 1) {
            curMonth.value = 12;
            curYear.value -= 1;
        } else {
            curMonth.value -= 1;
        }
        dateData.value = getMonthDate(curYear.value, curMonth.value);
        getData();
    };

    const getData = () => {
        isloading.value = true;
        let start = moment(`${curYear.value}-${curMonth.value}`)
            .startOf('month')
            .format('YYYY-MM-DD');
        let end = moment(`${curYear.value}-${curMonth.value}`).endOf('month').format('YYYY-MM-DD');

        axios
            .get({
                url: `/orderGroup?start=${start}&end=${end}`
            })
            .then((res: any) => {
                isloading.value = false;
                if (res.code == 200) {
                    result.value = res.data;
                    formatData();
                }
            });
    };

    const openModal = (item?: any) => {
        if (!item) {
            form.value.birthday = `${tYear.value}-${tMonth.value}-${tDay.value}`;
            visible.value = true;
        } else {
            let dateStr = `${item.year}-${item.month}-${item.showDate}`;
            let _date = Number(moment(dateStr).format('X'));

            if (_curDate >= _date && item.showDate == item.date) {
                form.value.birthday = `${item.year}-${item.month}-${item.showDate}`;
                visible.value = true;
            }
        }
    };

    const loginOut = () => {
        localStorage.clear();
        location.reload();
    };

    const save = () => {
        let { birthday, amount, type, purpose } = form.value;

        axios
            .post({
                url: `/order`,
                data: {
                    birthday: moment(birthday).format(),
                    type: parseInt(type),
                    amount: Number(amount),
                    purpose: purpose
                }
            })
            .then((res: any) => {
                if (res.code == 200) {
                    close();
                    LewMessage.success('记账成功');
                    getData();
                }
            });
    };

    const close = () => {
        form.value = {
            birthday: '',
            type: '1',
            amount: '',
            purpose: ''
        };
        visible.value = false;
    };

    onMounted(() => {
        getData();
    });
</script>
<template>
    <div class="wrapper">
        <div class="main">
            <lew-title style="color: #000; margin-bottom: 20px">
                {{ curYear }} 年 {{ curMonth }} 月
            </lew-title>

            <lew-flex mode="between" class="console">
                <lew-flex style="width: auto">
                    <lew-button type="normal" @click="prve"
                        ><ChevronLeft16Filled style="margin: 0px"
                    /></lew-button>
                    <lew-button type="normal" @click="next">
                        <ChevronRight16Filled style="margin: 0px"
                    /></lew-button>
                </lew-flex>
                <lew-flex style="width: auto">
                    <lew-button @click="openModal()">记一笔</lew-button>
                    <lew-button @click="loginOut" type="error">注销</lew-button>
                </lew-flex>
            </lew-flex>
            <div class="calendar" v-loading="{ isShow: isloading }">
                <div class="calendar-header">
                    <div v-for="(item, index) in getHeadDate" :key="index" class="item">
                        {{ item }}
                    </div>
                </div>
                <div class="calendar-date">
                    <div v-for="(item, index) in dateData" :key="index">
                        <lew-popover v-if="item?.expand?.order" trigger="hover">
                            <template #trigger>
                                <div :class="getClass(item)" @click="openModal(item)" class="item">
                                    <div class="show-date">
                                        {{ item.showDate }}
                                    </div>
                                    <div
                                        v-if="
                                            (item?.value || item?.status) &&
                                            item.showDate == item.date
                                        "
                                        class="money"
                                    >
                                        {{
                                            item?.expand?.total > 0
                                                ? '+' + item?.expand?.total
                                                : item?.expand?.total || item?.status
                                        }}
                                    </div>
                                </div>
                            </template>
                            <template #popover-body="{ hide }">
                                <div class="popover-body" style="width: 180px">
                                    <div class="title">{{ item?.expand?.date }}</div>
                                    <div
                                        v-for="(order, index) in item?.expand?.order"
                                        :key="index"
                                        class="popover-item"
                                        :class="`${
                                            order.type == 1 ? 'expenditure-item' : 'income-item'
                                        }`"
                                    >
                                        <div class="label">{{ order.purpose }}</div>
                                        <div class="value">￥{{ order.amount }}</div>
                                    </div>
                                    <br />
                                    <lew-button size="small" @click="hide(), openModal(item)"
                                        >加一笔</lew-button
                                    >
                                </div>
                            </template>
                        </lew-popover>

                        <div
                            v-if="!item?.expand?.order"
                            :class="getClass(item)"
                            @click="openModal(item)"
                            class="item"
                        >
                            <div class="show-date">
                                {{ item.showDate }}
                            </div>
                            <div
                                v-if="(item?.value || item?.status) && item.showDate == item.date"
                                class="money"
                            >
                                {{ item?.expand?.total || item?.status }}
                            </div>
                        </div>
                    </div>
                </div>
                <div class="footer">
                    <div>
                        <div class="label">月收入</div>
                        <div class="value">￥{{ result.income }}</div>
                    </div>
                    <div>
                        <div class="label">月支出</div>
                        <div class="value">￥{{ result.expenditure }}</div>
                    </div>
                    <div>
                        <div class="label">月盈亏</div>
                        <div class="value">￥{{ result.profit_loss }}</div>
                    </div>
                </div>
            </div>
        </div>
        <lew-modal :visible="visible" width="300px">
            <div class="modal-body">
                <lew-title :bold="700" style="color: #000; margin-bottom: 20px"> 记一笔 </lew-title>

                <lew-form-item direction="y" title="">
                    <lew-tabs
                        round
                        style="width: 100%"
                        itemWidth="100%"
                        v-model="form.type"
                        :options="[
                            { label: '支出', value: '1' },
                            { label: '收入', value: '2' }
                        ]"
                    ></lew-tabs>
                </lew-form-item>

                <lew-form-item direction="x" title="日期">
                    <lew-date-picker v-model="form.birthday" />
                </lew-form-item>

                <lew-form-item direction="x" title="金额">
                    <lew-input type="number" v-model="form.amount" />
                </lew-form-item>

                <lew-form-item style="margin-bottom: 30px" direction="x" title="用途">
                    <lew-input v-model="form.purpose" type="textarea" placeholder="请输入用途" />
                </lew-form-item>

                <lew-flex x="end">
                    <lew-button type="normal" @click="close">关闭 </lew-button>
                    <lew-button @click="save">保存</lew-button>
                </lew-flex></div
            >
        </lew-modal>
    </div>
</template>
<style lang="scss">
    .wrapper {
        display: flex;
        justify-content: center;
        width: 100%;
        height: 100vh;
        overflow: hidden;
        user-select: none;

        .main {
            width: calc(80px * 6 + 160px);
            padding: 10px;
            box-sizing: border-box;
            margin: auto;
            .console {
                width: 100%;
                margin-bottom: 15px;
            }
            .calendar {
                width: 100%;
                background-color: #fff;
                box-shadow: 0px 10px 30px rgb(207, 207, 207);
                padding: 15px;
                box-sizing: border-box;
                border-radius: var(--lew-form-border-radius);
                overflow: hidden;

                .item {
                    display: flex;
                    flex-direction: column;
                    justify-content: center;
                    align-items: center;
                }
                .calendar-header {
                    display: flex;
                    justify-content: center;
                    margin-bottom: 6px;
                    font-size: 16px;
                    gap: 5px;
                    .item {
                        width: 80px;
                        height: 30px;
                        color: #999;
                    }
                }
                .calendar-date {
                    display: flex;
                    flex-wrap: wrap;
                    justify-content: center;
                    gap: 5px;
                    .item {
                        border-radius: var(--lew-form-border-radius);
                        gap: 5px;
                        width: 80px;
                        height: 80px;

                        .show-date {
                            font-size: 16px;
                        }
                        .money {
                            padding: 4px;
                            font-size: 14px;
                        }
                    }

                    .today-item {
                        background-color: rgba(236, 220, 0, 0.2);
                    }
                    .pass-item {
                        background-color: rgb(243, 240, 240);
                        transition: all 0.05s;
                        cursor: pointer;
                    }
                    .pass-item:hover {
                        background-color: rgb(225, 225, 225);
                    }
                    .pass-item:active {
                        background-color: rgb(214, 214, 214);
                    }

                    .not-range-month-item {
                        opacity: 0.2;
                        cursor: default;
                    }

                    .expenditure-item {
                        background-color: rgba(3, 189, 99, 0.2);
                        color: #000;
                        .money {
                            color: rgb(13, 109, 63);
                        }
                    }
                    .expenditure-item:hover {
                        background-color: rgba(3, 189, 99, 0.3);
                    }
                    .expenditure-item:active {
                        background-color: rgba(3, 189, 99, 0.4);
                    }

                    .income-item {
                        background-color: rgba(201, 0, 23, 0.2);
                        color: #000;
                        .money {
                            color: rgb(157, 0, 18);
                        }
                    }
                    .income-item:hover {
                        background-color: rgba(201, 0, 23, 0.3);
                    }
                    .income-item:active {
                        background-color: rgba(201, 0, 23, 0.4);
                    }
                }
            }
            .footer {
                display: flex;
                align-items: center;
                justify-content: flex-end;
                background-color: #eee;
                border-radius: var(--lew-form-border-radius);
                padding: 10px 20px;
                box-sizing: border-box;
                margin-top: 10px;
                > div {
                    display: flex;
                    flex-direction: column;
                    align-items: flex-end;
                    margin-left: 50px;
                    .label {
                        margin-bottom: 5px;
                        color: rgb(125, 125, 125);
                    }
                    .value {
                        font-weight: bold;
                    }
                }
            }
        }
    }
</style>
<style lang="scss">
    .modal-body {
        padding: 20px;
    }
    .popover-body {
        display: flex;
        flex-direction: column;
        padding: 5px;
        .title {
            margin-bottom: 5px;
            font-weight: bold;
        }
        .popover-item {
            display: flex;
            justify-content: space-between;
            align-items: center;
            border-bottom: 1px #eee solid;
            padding: 8px 0px;

            .value {
                background-color: rgb(212, 212, 212);
                padding: 2px 8px;
                border-radius: 2px;
            }
        }

        .expenditure-item {
            .value {
                background-color: rgba(3, 189, 99, 0.2);
                padding: 0px 4px;
                border-radius: 2px;
            }
        }

        .income-item {
            .value {
                background-color: rgba(201, 0, 23, 0.2);
                padding: 0px 4px;
                border-radius: 2px;
            }
        }
        .popover-item:last-child {
            border-bottom: none;
        }
    }
</style>
