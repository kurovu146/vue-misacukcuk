<template>
    <div class="misa-dropdown">
        <div class="form-item">
            <label v-if="label">{{label}}</label>
            <div
                class="box"
                :class="{'border-red': error}"
            >
                <input
                    type="text"
                    :tabindex="tabindex"
                    @input="onChange"
                    v-model="search"
                    @keydown.down="onArrowDown"
                    @keydown.up="onArrowUp"
                    @keydown.tab="onTab"
                    @click="showOptions"
                    @keydown.enter="onEnter"
                    @blur="onBlur"
                    @focus="onFocus"
                    ref="input"
                    v-bind="$attrs"
                />
                <div
                    class="dropdown-icon"
                    @click="showOptions"
                >
                    <font-awesome-icon
                        v-if="!isOpen"
                        icon="angle-down"
                    />
                    <font-awesome-icon
                        v-else
                        icon="angle-up"
                    />
                </div>
                <ul
                    id="dropdown-options"
                    v-show="isOpen"
                    style="display: none;"
                    class="dropdown-options"
                >
                    <li
                        v-for="(option, i) in options"
                        :key="option.value"
                        @click="setResult(option)"
                        class="dropdown-result"
                        :class="{ 'is-active': result && result.text === option.text,'current-select': i === arrowCounter }"
                    >
                        {{ option.text }}
                    </li>
                </ul>
            </div>
        </div>
    </div>
</template>

<script>

/**
 * combobox
 * CreatedBy: Vũ Đức Tuấn 02/08/2921
 */
export default {
    name: "BaseDropdown",
    $emit: ["result"],
    props: {
        // là 1 array object {value: string, text: string}
        items: {
            type: Array,
            required: false,
            default: () => []
        },
        label: String,
        tabindex: String,
        defaultItem: {
            type: Object,
            default: () => null
        }
    },
    data() {
        return {
            isOpen: false,
            options: this.items,
            search: this.defaultItem?.text || this.items[0]?.text,
            result: this.defaultItem || this.items[0],
            arrowCounter: -1,
            error: false
        };
    },
    watch: {
        result() {
            this.$emit("result", this.result);
        }
    },
    // Lắng nghe sự kiện click ra bên ngoài combobox
    mounted() {
        document.addEventListener("click", this.handleClickOutside);
    },
    // xóa sự kiện này khi thoát khỏi xóa component
    destroyed() {
        document.removeEventListener("click", this.handleClickOutside);
    },
    methods: {
        // khi người dùng ấn vào 1 kết quả
        setResult(option) {
            this.search = option.text;
            this.isOpen = false;
            this.result = option;

            // chắc chắn người dùng đã chọn
            this.error = false;
        },
        // khi người dùng nhập thì sẽ bắt đầu lọc
        onChange() {
            this.filterOptions();
        },

        showOptions() {
            this.error = false;
            this.arrowCounter = -1;
            this.options = this.items;
            this.isOpen = !this.isOpen;

            // focus vào input khi click vào icon
            this.$refs.input.focus();
        },

        // phương thức khi người dùng click ra bên ngoài combobox
        handleClickOutside(event) {
            if (!this.$el.contains(event.target)) {
                this.isOpen = false;
                this.arrowCounter = -1;
            }
        },

        // sự kiện khi người dùng ấn mũi tên xuống
        onArrowDown() {
            if (this.arrowCounter < this.options.length - 1) {
                this.arrowCounter = this.arrowCounter + 1;
            } else {
                this.arrowCounter = 0;
            }
        },
        // sự kiện khi người dùng ấn mũi tên lên
        onArrowUp() {
            if (this.arrowCounter > 0) {
                this.arrowCounter = this.arrowCounter - 1;
            } else {
                this.arrowCounter = this.options.length - 1;
            }
        },
        // sự kiện khi người dùng ấn enter
        onEnter() {
            if (this.isOpen && this.options.length !== 0 && this.arrowCounter !== -1) {
                this.search = this.options[this.arrowCounter].text;
                this.result = this.options[this.arrowCounter];
                this.isOpen = false;
                this.arrowCounter = -1;

                // chắc chắn người dùng đã chọn
                this.error = false;
            }
        },

        onTab() {
            if (this.arrowCounter !== -1) {
                this.search = this.options[this.arrowCounter].text;
                this.result = this.options[this.arrowCounter];
                this.isOpen = false;
                this.arrowCounter = -1;

                // chắc chắn người dùng đã chọn
                this.error = false;
            } else {
                this.isOpen = false;
            }
        },

        onBlur() {
            const indexItem = this.items.findIndex(item => item.text === this.search);
            if (indexItem > -1) {
                this.result = this.items[indexItem];
                this.error = false;
            } else {
                this.error = true;
            }
            if (!this.result) {
                this.error = true;
            }
        },

        onFocus() {
            this.isOpen = true;
        }
    }
};
</script>

<style scoped>
.misa-dropdown {
    height: 40px;
    color: #000;
    display: flex;
    align-items: center;
    min-width: 200px;
    position: relative;
    cursor: pointer;
    background-color: #fff;
}

.misa-dropdown .dropdown-text {
    line-height: 40px;
    margin-left: 16px;
}

.misa-dropdown .arrow-icon {
    margin: 0px 12px;
}

.misa-dropdown .dropdown-list {
    position: absolute;
    width: 100%;
    top: 41px;
    z-index: 9999;
    background-color: #fff;
    box-shadow: 0 0px 0px 0 rgba(0, 0, 0, 0.2), 0 3px 4px 0 rgba(0, 0, 0, 0.19);
}

.misa-dropdown .dropdown-list .dropdown-item {
    height: 40px;
    color: #000;
    font-size: 13px;
    line-height: 40px;
    padding: 0px 16px;
    border: 1px;
    cursor: pointer;
}

.misa-dropdown .dropdown-list .dropdown-item:hover {
    background-color: #E9EBEE;
}

.misa-dropdown .dropdown-list .selected {
    background-color: #019160;
    color: #fff;
}

.misa-dropdown .dropdown-list .selected:hover {
    background-color: #019160;
    color: #fff;
}

.arrow {
    border: solid black;
    border-width: 0 2px 2px 0;
    display: inline-block;
    padding: 4px;
}

.right {
    transform: rotate(-45deg);
    -webkit-transform: rotate(-45deg);
}

.left {
    transform: rotate(135deg);
    -webkit-transform: rotate(135deg);
}

.up {
    transform: rotate(-135deg);
    -webkit-transform: rotate(-135deg);
    transition: all 0.4s;
}

.down {
    transform: rotate(45deg);
    -webkit-transform: rotate(45deg);
    transition: all 0.4s;
}
</style>
