<style lang="less">
    @import '../../styles/common.less';
</style>

<template>
    <div @keyup.enter="getData()">

        <Card>
            <Row>
                <Col span="4"> 关键词:
                <Input v-model="form.$key" placeholder="物流单/订单号/会员号" style="width: 150px" />
                </Col>

                <Col span="4"> 线路:
                <selectRoute @on-change="selectRouteChange"></selectRoute>
                </Col>
                <Col span="4"> 仓库:
                <selectWarhours @on-change="selectWarhoursChange"></selectWarhours>
                </Col>
                <Col span="4"> 扫描业务员:
                <selectDelivery @on-change="selectDeliveryChange"></selectDelivery>
                </Col>
                <Col span="4"> 订单状态:
                <selectStatu @on-change="selectStatuChange"></selectStatu>
                </Col>
                <Col span="4"> 第三方公司:
                <selectThirdPlatform @on-change="selectThirdPlatformChange"></selectThirdPlatform>
                </Col>
            </Row>
            <Row class="margin-top-10">
                <Col span="4"> 项目产品:
                <selectQitems @on-change="selectQitemsChange"></selectQitems>
                </Col>

                <Col span="12"> 时间日期:
                <datePickerCompoent @typeChange="typeChange" @startDateChange="startDateChange" @endDateChange="endDateChange"></datePickerCompoent>
                </Col>

            </Row>
            <Row class="margin-top-10">

                <Col span="3" style="padding-top: 6px;"> 表格样式
                <i-switch v-model="showControl" size="large"></i-switch>
                </Col>
                <Col span="3">
                <span style="margin: 0 10px;"><Button type="primary" icon="search" @click="getData()">搜索</Button></span>
                </Col>

            </Row>
        </Card>

        <Card style="position: relative;" class="margin-top-10">
            <initDataGridCompoent ref="dataGrid" :param="form" :autoWidth="false" :showControl="showControl" @selectRow="selectRow" @dbClickRow="dbClickRow"></initDataGridCompoent>
        </Card>
    </div>
</template>
<!-- 快递订单列表 -->
<script>
    import Util from '../../libs/util';
    import Cookies from 'js-cookie';
    // 头部组件
    import selectCollection from '../toolbar-components/selectCollection-component';
    import selectRoute from '../toolbar-components/selectRoute-component';
    import selectWarhours from '../toolbar-components/selectWarhours-component';
    import selectDelivery from '../toolbar-components/selectDelivery-component';
    import selectStatu from '../toolbar-components/selectStatu-component';
    import selectThirdPlatform from '../toolbar-components/selectThirdPlatform-component';
    import selectQitems from '../toolbar-components/selectQitems-component';
    import datePickerCompoent from '../toolbar-components/datePicker-component';

    // dataGrid组件
    import initDataGridCompoent from '../toolbar-components/initDataGrid-component';

    export default {
        name: 'QuerySuggestOrderView',
        components: {
            datePickerCompoent,
            selectCollection,
            selectRoute,
            selectWarhours,
            selectDelivery,
            selectStatu,
            selectThirdPlatform,
            selectQitems,

            initDataGridCompoent
        },
        data() {
            return {
                form: {
                    $key: '',
                    $StartDate: '',
                    $EndDate: '',

                    $IsExpress: '0',

                    isLoad: 'true',
                    $pageRecord: 15,
                    $rowIndex: 1,
                    strMethod: 'QuerySuggestOrderList',

                    $Status: '',
                    $LinCode: '',
                    $WarhoursCode: '',
                    $DistributorId: '',
                    $ThirdPlatformId: '',
                    $Qitems: '',
                    $IsCollection: '',
                    $rId :'',
                    $MStartDate: '',
                    $MEndDate: ''
                },
                showControl: false,
                selectRowData: undefined,
            };
        },
        methods: {
            getData() {
                var userInfo = JSON.parse(Cookies.get('userInfo'));
                if(userInfo.RoleId == 21) //代理商为21
                {
                    var par = {
                        strAddress: userInfo.address
                    }
                    Util.getAjaxData(par, undefined, '18c6f7e0-ef20-4088-b034-50c1ef577e39').then((res) => {
                        this.form.$rId = res.Code;
                        this.firstGetData()
                    })
                } else
                    this.firstGetData()

            },
            firstGetData() {
                for(var key in this.form) {
                    if(this.form[key] === '') {
                        this.form[key] = undefined;
                    }
                }
                this.form.$rowIndex = 1;
                this.$refs.dataGrid.initDataGrid()
            },
            selectRow(data) {
                this.selectRowData = data
            },
            dbClickRow(data) {
                this.selectRowData = data
            },
            startDateChange(data) {
                this.form.$StartDate = data
                this.form.$MStartDate = data
            },
            endDateChange(data) {
                this.form.$EndDate = data
                this.form.$MEndDate = data
            },
            selectCollectionChange(data) {
                this.form.$IsCollection = data
            },

            selectRouteChange(data) {
                this.form.$LinCode = data
            },
            selectWarhoursChange(data) {
                this.form.$WarhoursCode = data
            },
            selectDeliveryChange(data) {
                this.form.$DistributorId = data
            },
            selectStatuChange(data) {
                this.form.$Statu = data
            },
            selectThirdPlatformChange(data) {
                this.form.$ThirdPlatformId = data
            },
            selectQitemsChange(data) {
                this.form.$Qitems = data
            },
            typeChange(data) {
                // 根据数值类型修改
                if(data == 0) {

                    this.form.$MStartDate = undefined
                    this.form.$MEndDate = undefined
                }
                if(data == 1) {
                    this.form.$EndDate = undefined
                    this.form.$StartDate = undefined
                }
            }
        },
        mounted() {
            // var date = Util.nowTime(this);
            // this.form.$StartDate = date.forweek;
            // this.form.$EndDate = date.formatnowdate;
            this.getData()
        }
    };
</script>
