# Angular Project EP3 : LOGIN/OUT REGISTER INTERFACE

## Create Form
- New file 'register.interface.ts'
- Look all function on a page
  ```bash
  import { FormGroup } from "@angular/forms"
  export interface IRegisterComponent {
      form: FormGroup;
      Url: any;
      
      onSubmit();
    }
  ```
- instead on 'register.componente.ts'
  ```bash
  export class RegisterComponent implements IRegisterComponent {

    constructor(
      private builder: FormBuilder
    ) {
      this.initialCreateFormData();
    }

    //Form Create
    private initialCreateFormData() {
      this.form = this.builder.group({
        firstname: [],
        lastname: [],
        username: [],
        password: [],
        repassword: [],
      });
    }
    Url = AppURL;
    form: FormGroup;

    //Register
    onSubmit() {
      console.log(this.form.value);
    }
  }

  ```
- on Modules.ts
  ```bash
  imports:[ FormsModule, ReactiveFormsModule]
  exports:[ FormsModule, ReactiveFormsModule]
  ```
- on 'register.component.html'
  ```bash
  <form class="login-form" [formGroup]='form' (submit)="onSubmit()">
  <input class="form-control" formControlName='firstname' type="text" placeholder="Firstname">
  ```


## Validator
- add ['',[Validators.required]] to 'register.components.ts'
  ```bash
  private initialCreateFormData() {
    this.form = this.builder.group({
      firstname: ['',[Validators.required]],
      lastname: ['',[Validators.required]],
      username: ['',[Validators.required]],
      password: ['',[Validators.required]],
      repassword: ['',[Validators.required]],
    });
  }
  ```

## Create Alert Service
- add new folder 'services' on 'shared'
- create new file alert.services.ts
  ```bash
  import { Injectable } from "@angular/core";

  declare const swal: any;
  @Injectable()
  export class AlertService {
      notify() {
          swal("Error, Somethins Wrong", "Please fill in all required fields.", "error");
      }
  }   
  ```
- on 'shared.module.ts' add provider
  ```bash
    providers: [ AlertService ]
  ```
- on 'register.component.ts' add to Constructor
  ```bash
  private alertService : AlertService
  ```
- on 'register.component.ts' add to onSubmit()
  ```bash
  onSubmit() {
    if (this.form.invalid) {
      this.alertService.notify();
    }
    console.log(this.form.value);
  }
  ```
## Add Validate and Manual Validate
- Manual Validator on 'register.component.ts' 
  ```bash
    //Validator
    password: ['', [Validators.required, Validator.pattern($&&*!(@&))]],
    //Manual Validator
    repassword: ['', [Validators.required, this.comparePassword('password')]],
  ```
  ```bash
  // Match Password Validator
  private comparePassword(inputPassword: string) {
      return function (confirmPassword: AbstractControl) {
          if (!confirmPassword.parent) return;
          console.log("C PASSWORD ");
          console.log(confirmPassword);

          const password = confirmPassword.parent.get(inputPassword);
          const passwordSubcribe = password.valueChanges.subscribe(() => {
              confirmPassword.updateValueAndValidity();
          });
          
          if (confirmPassword.value === password.value) return;

          return { compare: true };
      }
  }
  ```
## Account Service
- new file on share/service 'account.service.ts'
  ```bash
@Injectable()
export class AccountService{
    onRegister(model: IRegister){
        return new Promise((resolve,reject) => {
            resolve(model); 
        });
    }
}
  ```
- 'register.interface.ts
  ```bash
  export interface IRegister{
      firstname: string;
      lastname: string;
      username: string;
      password: string;
      repassword: string;
  }
  ```
- 'register.component.ts
  ```bash
  constructor{
        private accountService: AccountService,
        private router: Router, 
  ```
  ```bash
  onSubmit() {
    if (this.form.invalid) {
        this.alertService.notify("Please fill in all required fields.");
    }
    // tranfer data to server
    this.accountService
        .onRegister(this.form.value)
        .then(res => {
            this.router.navigate(['/', AppURL.Login])
        })
        .catch(err => this.alertService.notify(err.Message));
  }
  ```

## set Token front END
- create new folder app services.
- create new file 'authen.services.ts'
  ```bash
  import { Injectable } from "@angular/core";

  @Injectable({
      providedIn: 'root'
  })
  export class AuthenService{
      private accessKey = 'accessToken';

      // set access token on browser memory
      setAuthenticated(accessToken: string){
          localStorage.setItem(this.accessKey, accessToken)
      }
      // get access token from browser memory
      getAuthenticated(): string{
          return localStorage.getItem(this.accessKey);
      }
      // clear access token from browser memory
      clearAuthenticated(){
          localStorage.removeItem(this.accessKey)
      }
  }
  ```
- on 'account.service.ts'
  ```bash
  return new Promise<{ accessToken: string }>
  ```
- on 'login.component.ts'
  ```bash
  this.accountService
    .onLogin(this.form.value)
    .then(res => {
      // get session
      this.authenService.setAuthenticated(res.accessToken);
  ```