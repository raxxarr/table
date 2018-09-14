<template>
    <div class="table-page">
        <div class="table-container">
            <!-- mask -->
            <table class="table table-mask">
                <tbody><tr><td ref="maskCell"></td></tr></tbody>
            </table>
            <!-- fixed head -->
            <table class="table table-head" ref="tableHead">
                <thead>
                    <tr>
                        <th v-for="key in headKeys" :key="key"><span ref="tableHeadTh" class="table-head-cell">{{ key }}</span></th>
                    </tr>
                </thead>
                <!-- placeholder -->
                <tbody>
                    <tr>
                        <td v-for="(val, key) in tableData[0]" :key="key"><span class="placeholder-cell">{{ val }}</span></td>
                    </tr>
                </tbody>
            </table>
            <!-- fixed column -->
            <table class="table table-column" ref="tableColumn">
                <thead>
                    <tr>
                        <th>{{ headKeys[0] }}</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="row in tableData" :key="row.id" ref="tableColumnTr">
                        <td>{{ row[headKeys[0]] }}</td>
                    </tr>
                </tbody>
            </table>
            <!-- content -->
            <div @scroll="contentScroll" class="table-main-wrap">
                <table class="table table-main" ref="tableMain">
                    <thead>
                        <tr class="table-main-placeholder-tr">
                            <th v-for="key in headKeys" :key="key" ref="tableMainTh"><span class="table-main-placeholder-cell">{{ key }}</span></th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-for="row in tableData" :key="row.id" ref="tableMainTr">
                            <!-- { id: 0, A: 1 } -->
                            <td v-for="(val, key) in row" :key="key">{{ val }}</td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>
        <!-- config -->
        <div class="config-dialog" v-show="dialogVisible">
            <div class="config-inline-wrap">
                <span>行数：</span><input type="text" v-model.trim="count">
            </div>
            <div class="config-inline-wrap">
                <span>列数：</span><input type="text" v-model.trim="headCount">
            </div>
            <div class="ok-btn" @click="update">{{ loading ? '请稍等' : '确定' }}</div>
        </div>
        <!-- config btn -->
        <div class="config-btn" @click="showDialog">+</div>
    </div>
</template>
<script>
export default {
    name: 'TablePage',

    data () {
        return {
            tableMain: null,
            tableColumn: null,
            tableHead: null,
            count: 100,
            headCount: 100,
            dialogVisible: false,
            tableData: []
        }
    },

    computed: {
        headKeys () {
            return Object.keys(this.tableData[0])
        }
    },

    methods: {
        contentScroll (e) {
            if (!this.tableMain) { return }
            const FIX = 3

            const pos = this.tableMain.getBoundingClientRect()
            const top = pos.top + FIX <= 0 ? pos.top + FIX : 0
            this.tableColumn.style.top = top + 'px'
            this.tableHead.style.left = pos.left + 'px'
        },

        computeRowHeight () {
            // 拿到 maintable tbody 首行，计算行高，给当前的 column
            const tableMainTrs = this.$refs.tableMainTr
            const tableColumnTrs = this.$refs.tableColumnTr
            tableMainTrs.forEach((cell, index) => {
                tableColumnTrs[index].style.height = cell.offsetHeight + 'px'
            })
        },

        computedHeadWidth () {
            const BORDER_WIDTH = 1
            // 等 table 渲染完毕，拿到 maintable thead 首行，计算每个 cell 宽度，给当前列的 head
            const tableHeadThs = this.$refs.tableHeadTh
            const tableMainThs = this.$refs.tableMainTh
            const tableMaskCell = this.$refs.maskCell
            tableMainThs.forEach((cell, index) => {
                if (index === 0) {
                    tableMaskCell.style.width = cell.offsetWidth - BORDER_WIDTH + 'px'
                }
                tableHeadThs[index].style.width = cell.offsetWidth - BORDER_WIDTH + 'px'
            })
        },

        genData (count, headCount) {
            const alphabet = this.genAlphabet()
            const res = []
            for (let i = 0; i < count; i++) {
                let obj = {}
                obj.id = i
                for (let _i = 0; _i < headCount; _i++) {
                    const prefix = alphabet[parseInt(_i / 26) - 1] || ''
                    const main = alphabet[_i % 26]
                    const name = prefix + main
                    if (_i === 3) {
                        obj[name] = `33333333
                        3333`
                    } else {
                        obj[name] = _i
                    }
                }
                res.push(obj)
            }
            return res
        },

        genAlphabet () {
            let str = []
            for (let i = 65; i < 91; i++) {
                str.push(String.fromCharCode(i))
            }
            return str
        },

        getDoms () {
            this.tableMain = this.$refs.tableMain
            this.tableColumn = this.$refs.tableColumn
            this.tableHead = this.$refs.tableHead
        },

        update () {
            if (this.loading) { return }
            const count = Number(this.count || 0)
            const headCount = Number(this.headCount || 0)
            this.tableData = this.genData(count, headCount)

            this.$nextTick(() => {
                this.getDoms()
                this.computeRowHeight()
                this.computedHeadWidth()
            })

            this.loading = false
            this.dialogVisible = false
        },

        showDialog () {
            this.dialogVisible = true
        }
    },

    created () {
        this.update()
    }
}
</script>
<style lang="less" scoped>
@borderColor: #ddd;
.table-page {
    overflow: hidden;
    height: 100%;
}

.table-container {
    position: relative;
    width: 100%;
    height: 100%;
}
.table {
    // width: 100%;
    // table-layout: fixed;
    border: 1px solid @borderColor;
    border-collapse: collapse;

    th,
    td {
        border: 1px solid @borderColor;
    }
}

// ------- fixed head -------

.table-head {
    position: relative;

    tbody {
        visibility: hidden;

        td {
            padding: 0;
            overflow: hidden;
        }
    }

    tr {
        height: 30px;
        background-color: #eee;
        overflow: hidden;
    }

    th {
        padding: 0;
    }

    .placeholder-cell {
        // white-space: nowrap;
        height: 30px;
        overflow: hidden;
        float: left;
    }

    .table-head-cell {
        display: inline-block;
    }
}

// ------- fixed column -------

.table-column {
    position: absolute;
    top: 0;
    left: 0;
    background-color: #eee;

    thead tr {
        height: 30px;
    }

    th,
    td {
        text-align: center;
    }
}

// ------- main table -------

.table-main-wrap {
    // head table 的隐藏行高度为 30px；main table 隐藏行的高度为 30px;
    margin-top: -60px;
    height: 100%;
    overflow-x: auto;
    overflow-y: auto;
}

.table-main {
    // header table 隐藏行的上下 border, main table 隐藏行的上下 border
    margin-top: -5px;

    table-main-placeholder-tr {
        overflow: hidden;
    }

    .table-main-placeholder-cell {
        float: left;
        height: 30px;
        line-height: 30px;
        overflow: hidden;
    }
}

// ------- fixed mask -------

.table-mask {
    position: absolute;
    top: 0;
    left: 0;
    height: 31px;
    background: #eee;
    z-index: 10;

    td {
        padding: 0;
        margin: 0;
        border: none;
    }
}

tr, td {
    padding: 0 10px;
}

// ------- config dialog -------
@textColor: #999;
.config-dialog {
    position: fixed;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    padding: 20px;
    width: 200px;
    height: 130px;
    background-color: #f3f3f3;
    box-shadow: #ececec 1px 1px 3px 1px;

    // layout
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
}

.config-inline-wrap {
    margin-bottom: 20px;
    font-size: 14px;
    color: @textColor;
    font-weight: 200;

    input {
        outline: none;
        border: none;
        margin: 0;
        color: @textColor;
    }
}

.config-btn {
    position: fixed;
    right: 20px;
    bottom: 20px;
    width: 50px;
    height: 50px;
    padding: 0;
    border-radius: 50%;
    background-color: #f3f3f3;
    font-size: 20px;
    color: @textColor;

    display: flex;
    align-items: center;
    justify-content: center;
}

.config-btn:active {
    background-color: #e0e0e0;
}

.ok-btn {
    font-size: 14px;
    color: @textColor;
}

.ok-btn:active {
    color: #000;
}
</style>
