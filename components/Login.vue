<template>
  <div class="">
    <!-- Login -->
    <a-row type="flex" justify="center" v-if="isLoginMode">
      <a-col
        :xs="{ span: 22 }"
        :sm="{ span: 20 }"
        :md="{ span: 10 }"
        :lg="{ span: 8 }"
        class="main-container"
      >
        <div class="w-100 login-forms">
          <h1>Login</h1>
          <a-form layout="vertical" :form="form" @submit="handleSubmit">
            <a-form-item
              :validate-status="userNameError() ? 'error' : ''"
              :help="userNameError() || ''"
            >
              <a-input
                v-decorator="[
                  'userName',
                  {
                    rules: [
                      {
                        required: true,
                        message: 'Please input your username!',
                      },
                    ],
                  },
                ]"
                placeholder="Username"
              >
                <a-icon
                  slot="prefix"
                  type="user"
                  style="color: rgba(0, 0, 0, 0.85)"
                />
              </a-input>
            </a-form-item>
            <a-form-item
              :validate-status="passwordError() ? 'error' : ''"
              :help="passwordError() || ''"
            >
              <a-input
                v-decorator="[
                  'password',
                  {
                    rules: [
                      {
                        required: true,
                        message: 'Please input your Password!',
                      },
                    ],
                  },
                ]"
                type="password"
                placeholder="Password"
              >
                <a-icon
                  slot="prefix"
                  type="lock"
                  style="color: rgba(0, 0, 0, 0.85)"
                />
              </a-input>
            </a-form-item>
            <a-form-item>
              <div class="forgotpass">
                <a-checkbox> Remember me </a-checkbox>
                <a class="login-form-forgot" href=""> Forgot password ?</a>
              </div>
            </a-form-item>
            <a-form-item>
              <a-button type="primary" html-type="submit" block>
                Log in
              </a-button>
            </a-form-item>
            <a-form-item>
              <a href="#" v-on:click="changeMode()">Register now</a>
              <!-- <Nuxt-link to="/register">Register now</Nuxt-link> -->
            </a-form-item>
          </a-form>
        </div>
      </a-col>
    </a-row>
    <!-- Register  -->
    <a-row type="flex" justify="center" v-else-if="!isLoginMode">
      <a-col
        :xs="{ span: 22 }"
        :sm="{ span: 20 }"
        :md="{ span: 8 }"
        class="main-container"
      >
        <h1>Register</h1>
        <a-form :form="formRegister" @submit="handleSubmitRegister">
          <a-form-item label="Username">
            <a-input
              v-decorator="[
                'userName',
                {
                  rules: [
                    {
                      required: true,
                      message: 'Please input your user name!',
                    },
                  ],
                },
              ]"
            />
          </a-form-item>
          <a-form-item label="Password" has-feedback>
            <a-input
              v-decorator="[
                'password',
                {
                  rules: [
                    {
                      required: true,
                      message: 'Please input your password!',
                    },
                    {
                      validator: validateToNextPassword,
                    },
                  ],
                },
              ]"
              type="password"
            />
          </a-form-item>
          <a-form-item label="Confirm Password" has-feedback>
            <a-input
              v-decorator="[
                'confirm',
                {
                  rules: [
                    {
                      required: true,
                      message: 'Please confirm your password!',
                    },
                    {
                      validator: compareToFirstPassword,
                    },
                  ],
                },
              ]"
              type="password"
              @blur="handleConfirmBlur"
            />
          </a-form-item>
          <a-form-item>
            <a-button type="primary" html-type="submit" block>
              Register
            </a-button>
          </a-form-item>
          <a-form-item>
            <a href="#" v-on:click="changeMode()">Back to Login</a>
          </a-form-item>
        </a-form>
      </a-col>
    </a-row>
  </div>
</template>

<script>
import axios from "axios";

function hasErrors(fieldsError) {
  return Object.keys(fieldsError).some((field) => fieldsError[field]);
}

export default {
  data() {
    return {
      // Login
      hasErrors,
      form: this.$form.createForm(this, { name: "horizontal_login" }),
      formRegister: this.$form.createForm(this, { name: "register" }),
      listUser: [],
      isLoginMode: true,
      // Register
      confirmDirty: false,
      autoCompleteResult: [],
      jsonData: null,
    };
  },
  mounted() {
    this.$nextTick(() => {
      // To disabled submit button at the beginning.
      this.form.validateFields();
    });
    this.getData();
  },
  beforeCreate() {
    this.formRegister = this.$form.createForm(this, { name: "register" });
  },
  methods: {
    // Only show error after a field is touched.
    userNameError() {
      const { getFieldError, isFieldTouched } = this.form;
      return isFieldTouched("userName") && getFieldError("userName");
    },
    // Only show error after a field is touched.
    passwordError() {
      const { getFieldError, isFieldTouched } = this.form;
      return isFieldTouched("password") && getFieldError("password");
    },
    handleSubmit(e) {
      e.preventDefault();
      this.form.validateFields((err, values) => {
        if (!err) {
          const user = this.listUser.find(
            (el) =>
              el.userName == values.userName && el.password == values.password
          );
          if (user) {
            this.$notification.open({
              message: "Đăng nhập thành công !",
              icon: <a-icon type="smile" style="color: #0bcd1b" />,
            });
            // this.$router.push({path:"/mainpage"})
          } else {
            this.$notification.open({
              message: "Tài khoản hoặc mật khẩu không đúng , vui lòng thử lại.",
              icon: <a-icon type="smile" style="color: #d51200" />,
            });
          }
        }
      });
    },
    async getData() {
      try {
        // Gửi yêu cầu GET để lấy dữ liệu từ tệp JSON
        const response = await axios.get("/data.json");

        // Lưu trữ dữ liệu vào biến jsonData trong data()
        this.listUser = response.data.lists;
        this.jsonData = response.data;

        window.localStorage.setItem("lists", JSON.stringify(response.data));
      } catch (error) {
        console.error("Đã xảy ra lỗi khi lấy dữ liệu:", error);
      }
    },
    // Registers
    handleSubmitRegister(e) {
      e.preventDefault();
      this.formRegister.validateFieldsAndScroll((err, values) => {
        const newUser = {
          userName: values.userName,
          password: values.password,
        };

        const checkUserName = this.listUser.find(
          (el) => el.userName == values.userName
        );
        if (!err && values.password == values.confirm && !checkUserName) {
          this.addNewUser(newUser);
          this.$notification.open({
            message: "Đăng ký thành công !",
            icon: <a-icon type="smile" style="color: #0bcd1b" />,
          });
        } else {
          this.$notification.open({
            message: "Tài khoản đã tồn tại !",
            icon: <a-icon type="smile" style="color: #d51200" />,
          });
        }
      });
    },
    handleConfirmBlur(e) {
      const value = e.target.value;
      this.confirmDirty = this.confirmDirty || !!value;
    },
    compareToFirstPassword(rule, value, callback) {
      const form = this.formRegister;
      if (value && value !== form.getFieldValue("password")) {
        callback("Two passwords that you enter is inconsistent!");
      } else {
        callback();
      }
    },
    validateToNextPassword(rule, value, callback) {
      const form = this.formRegister;
      if (value && this.confirmDirty) {
        form.validateFields(["confirm"], { force: true });
      }
      callback();
    },
    async addNewUser(newData) {
      try {
        this.listUser.push(newData);
        this.jsonData.lists = this.listUser;
        window.localStorage.setItem("lists", JSON.stringify(this.jsonData));
      } catch (error) {
        console.error("Không push được", error);
      }
    },
    changeMode() {
      this.isLoginMode = !this.isLoginMode;
    },
  },
};
</script>

<style lang="css">
.login-forms {
  padding-top: 25px;
}

.forgotpass {
  width: 100%;
}
.login-form-forgot {
  float: right;
}
.main-container {
  padding: 35px;
  border: 1px solid black;
  border-radius: 10px;
  height: 75vh;
  margin-top: 10vh;
  background: white;
}
.alert-message {
  color: #d51200;
}
</style>
