<template>
    <div class="desc-header">
        <h4 class="desc-title">Description Form</h4>
        <p class="desc-subtitle">test components</p>
    </div>
    <div class="desc-body">
        <div>
            <input type="text" v-model.lazy="description" @keyup="onKeyUpClick($event)" id="description">
        </div>
        <div>
            <span class="error-balanced">{{ msg }}</span>
        </div>
    </div>
    <div class="desc-footer"></div>
</template>


<script>


export default {

    data() {
        return {
            description: '',
            isValidDescription: false,
            msg: 'The text is balanced.'
        }
    },
    computed: {

    },
    methods: {
        onKeyUpClick(e) {
            if (e.target.value && e.target.value.slice(-1) == '?') {
                this.isBalanced(e.target.value);
            }
        },

        isBalanced(val) {
            debugger;
            // this.isValidDescription = (/^(?:[^[\]()]*\((?:[^[\]()]*\[[^[\]()]*\][^[\]()]*)*\)(?:[^[\]()]*\{(?:[^[\]()]*\[(?:[^[\]()]*\([^[\]()]*\)[^[\]()]*)*\][^[\]()]*)*\}[^[\]()]*)*[^[\]()]*)*$/.test(val))
            this.isValidDescription = !this.BalanceCore(val);
            if (this.isValidDescription) {
                this.msg = 'The text is not balanced.';
            }
            else {
                this.msg = 'The text is balanced.';
            }
        },
        BalanceCore(expression) {
            const stack = [];
            const pairs = {
                '(': ')',
                '[': ']',
                '{': '}'
            };

            for (let char of expression) {
                if (char === '(' || char === '[' || char === '{') {
                    // اگر کاراکتر اول پرانتز باز شده باشد، آن را به پشته اضافه میکنه.
                    stack.push(char);
                } else if (char === ')' || char === ']' || char === '}') {
                    // اگر کاراکتر یکی از پرانتزهای بسته شده باشد،
                    // بررسی میکنه که آیا پشته خالی است یا نه.
                    if (stack.length === 0) {
                        return false; // اگر پشته خالی باشد، ترتیب اشتباهه .
                    }

                    // بررسی میکنه که آیا کاراکتر قرینه پرانتز باز شده است یا نه.
                    const top = stack.pop();
                    if (pairs[top] !== char) {
                        return false; // اگر قرینه نباشد، ترتیب اشتباه است.
                    }
                }
            }

            // در پایان، اگر پشته خالی باشد، تمام پرانتزها به ترتیب صحیح بسته شده‌اند.
            return stack.length === 0;
        }
   
    },
    watch: {

    }
}
</script>
<style lang="scss" scoped>
.error-balanced {
    color: red;
}
</style>