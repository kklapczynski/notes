# Angular component selectors.

In @Component selectors may be defined as in CSS:

    '[app-servers]' => ref. in html as <div app-servers></div>  (css attribute)
    '.app-servers' => ref. in html as <div class="app-servers"></div>  (css class)

    PS: id or pseudo selectors don't work like above with angular

# Angular syntax
## General
- __{{ }}__ string interpolation

   ```html
    <p>{{this.serverCreationStatus}}</p>
   ```
- __[ property ]__ = "ts expression"  - property binding

   ```html
    <button 
    class="btn btn-primary"
    [disabled]="!allowNewServers"
    >Add server</button>
   ```

- $event - object holding event data (po prostu odpowiednik obiektu 'event' w JS) for event binding
    ```html
    <input 
    type="text"
    class="form-control"
    (input)="onUpdateServerName($event)">
    ```
- __[(ngModel)]__  two-way-databinding

   ```html
    <input 
    type="text"
    class="form-control"
    [(ngModel)]="serverName">
   ```
## Directives
Directives are instructions in the DOM.
Components are directives with a template.
- _STRUCTURAL DIRECTIVES_: * marks structural directive - changes structure of DOM: adds or removes elements; cannot have 2 structural directives on element :

   - __*ngIf__

      ```html
      <p *ngIf="this.isServerCreated">Server >>> {{this.serverName}} <<< was created.</p>
      ```
      - __*ngIf__ with __else__ and __ng-template__ with __local reference__
      ```html
         <div class="col-md-7">
         <app-recipes-details 
               *ngIf="selectedRecipe; else infoText"
               [recipe]="selectedRecipe"
         ></app-recipes-details>
         <ng-template #infoText>
               <p>Please select recipe.</p>
         </ng-template>
      </div>
      ```
   - __*ngFor__ - loops through an array to add multiple elements

      ```html
      <app-server *ngFor="let server of servers"></app-server>
      <div *ngFor="let logItem of log; let i = index">{{ logItem }}</div>
      ```
   - __*ngSwitchCase__ and __*ngSwitchDefault__ use with __[ngSwitch]="case_value"__
      
      ```html
         <button class="btn btn-primary" (click)="changeValue(value)">Change value</button>
         <div [ngSwitch]="value">
            <p *ngSwitchCase="1">Value is {{ value }}</p>
            <p *ngSwitchCase="5">Value is {{ value }}</p>
            <p *ngSwitchCase="10">Value is {{ value }}</p>
            <p *ngSwitchCase="15">Value is {{ value }}</p>
            <p *ngSwitchCase="100">Value is {{ value }}</p>
            <p *ngSwitchDefault>Value is default</p>
         </div>
      ```

- __ngStyle__ - dynamically change style of elements

   ```html
    <p [ngStyle]="{'color': getColor(), 'background-color': getBackgroundColor()}">{{'Server'}} with ID {{serverID}} is {{ getServerStatus()}}</p>
   ```
- __ngClass__ - dynamically adds or removes css class to element

   ```html
    <p 
    [ngClass]="{'online': isOnline()}"
   ```

- __ng_content__ - hook in child component html to allow placing html content inside component tags ()of this child component) in html of parent component

   - https://www.udemy.com/course/the-complete-guide-to-angular-2/learn/lecture/6656100#bookmarks
   - child component: server-element
   ```html
      <div
         class="panel panel-default">
         <div class="panel-heading">{{ element.name }}</div>
         <div class="panel-body">
            <ng-content></ng-content>
            <!-- <p>
            <strong *ngIf="element.type === 'server'" style="color: red">{{ element.content }}</strong>
            <em *ngIf="element.type === 'blueprint'">{{ element.content }}</em>
            </p> -->
         </div>
      </div>
   ```
   - parent component: app
   ```html
      <div class="row">
         <div class="col-xs-12">
            <app-server-element 
            *ngFor="let element of serverElements"
            [srvElement]="element"
            >
            <p>
               <strong *ngIf="element.type === 'server'" style="color: red">{{ element.content }}</strong>
               <em *ngIf="element.type === 'blueprint'">{{ element.content }}</em>
            </p>
            </app-server-element>
         </div>
      </div>
   ```

- ### __Custom directives__
   ```ts
      import { Directive, Renderer2, OnInit, ElementRef } from '@angular/core';

      @Directive({
      selector: '[appBetterHighlight]'
      })
      export class BetterHighlightDirective implements OnInit {
         // better - more universal (for Service Workers e.g) approach is to not set style of a DOM element directly (as in basic-highlight), but using Renderer2
         constructor(private el: ElementRef, private renderer: Renderer2) { }

         ngOnInit() {
            this.renderer.setStyle(this.el.nativeElement, 'background-color', 'lightblue');
         }
      }
   ```
## Decorators

- __@Input(optional alias)__ - makes property of component accessible for parent component, that is instantiating component with property decorated with @Input; alias to be used for property binding in parent element; parent component can bind to child component property and change it = assign value to it

   ```TS
      @Component({
         selector: 'app-server-element',
         templateUrl: './server-element.component.html',
         styleUrls: ['./server-element.component.css']
         })
         export class ServerElementComponent implements OnInit {
         @Input('srvElement') element: {name: string, type: string, content: string};

         constructor() { }

         ngOnInit() {
         }

         }
   ```
   ```html
      <div class="col-xs-12">
         <app-server-element 
         *ngFor="let element of serverElements"
         [srvElement]="element"
         ></app-server-element>
      </div>
   ```

- __@Output(optional alias)__ - makes property of child component emittable = parent component can bind to __CUSTOM EVENT__ emitted from child component

   ```TS
      export class CockpitComponent implements OnInit {  // child component
         @Output() serverCreated = new EventEmitter<{serverName: string, serverContent: string}>();
         @Output('bpc') blueprintCreated = new EventEmitter<{serverName: string, serverContent: string}>();

         newServerName = '';
         newServerContent = '';
         

         onAddServer() {
            this.serverCreated.emit({serverName: this.newServerName, serverContent: this.newServerContent});
         }

         onAddBlueprint() {
            this.blueprintCreated.emit({serverName: this.newServerName, serverContent: this.newServerContent});
         }
   ```
   app.component.html - parent component
   ```html
      <div class="row">
         <div class="col-xs-12">
            <app-cockpit 
            (serverCreated)="onServerAdded($event)"
            (bpc)="onBlueprintAdded($event)"
            ></app-cockpit>
         </div>
      </div>
   ```
- __@ViewChild()__ - returns ElementRef with nativeElement property; lets us get html el. by local ref. or Component name to use it in TS code

   - https://www.udemy.com/course/the-complete-guide-to-angular-2/learn/lecture/6656094#bookmarks

   ```TS
      @ViewChild('serverContentInput', {static: true}) serverContentInput: ElementRef;  // 1. arg: local reference or component name; {static: true} - nedded to use serverContentInput in ngOnInit()

      onAddServer(serverNameInput: HTMLInputElement) {
         // console.log(this.serverContentInput.nativeElement.value);
         this.serverCreated.emit({serverName: serverNameInput.value, serverContent: this.serverContentInput.nativeElement.value});
  }
   ```

   ```html
      <input 
         type="text"
         class="form-control"
         #serverContentInput>
   ```
- __@ContentChild()__ - gives access to content (of child component) added in parent component
   - https://www.udemy.com/course/the-complete-guide-to-angular-2/learn/lecture/6656114#bookmarks
   - server-element.component.ts - child component of app.component
   ```TS
      @ContentChild('paragraphContent', {static: false}) paragraph: ElementRef; // gives access to content of child component added in parent component => local reference 'paragraphContent' is defined in app.component.html and is not available for @ViewChild()
   ```
   - app.component.html
   ```html
      <app-server-element 
        *ngFor="let element of serverElements"
        [srvElement]="element"
        [name]="element.name"
      >
        <p #paragraphContent>
          <strong *ngIf="element.type === 'server'" style="color: red">{{ element.content }}</strong>
          <em *ngIf="element.type === 'blueprint'">{{ element.content }}</em>
        </p>
      </app-server-element>
   ```
- __@HostBinding__ bind directive property with property of host element = el. with directive as an attribute
- __@HostListener__ - sets up event listener on elem.
   ```ts
      import { Directive, Renderer2, OnInit, ElementRef, HostListener, HostBinding } from '@angular/core';

   @Directive({
   selector: '[appBetterHighlight]'
   })
   export class BetterHighlightDirective implements OnInit{
      // better - more universal (for Service Workers e.g) approach is to not set style of a DOM element directly (as in basic-highlight), but using Renderer2
      // constructor(private el: ElementRef, private renderer: Renderer2) { }

      ngOnInit() {
         // this.renderer.setStyle(this.el.nativeElement, 'background-color', 'lightblue');
      }

      @HostBinding('style.backgroundColor') backgroundColor: string = 'transparent';

      // add interactivity: chnage background-color when hovering over element, that has directive as an attribute
      @HostListener('mousemove') onMouseEnter(eventData: Event) {
         // this.renderer.setStyle(this.el.nativeElement, 'background-color', 'lightblue');
         this.backgroundColor = 'lightblue';
      }

      @HostListener('mouseout') onMouseLeave(eventData: Event) {
         // this.renderer.setStyle(this.el.nativeElement, 'background-color', 'transparent');
         this.backgroundColor = 'transparent';
      }
   }

   ```


## @Component properties

- encapsulation : styles separation per component

```TS
   import { Component, OnInit, Input, ViewEncapsulation } from '@angular/core';

   @Component({
   selector: 'app-server-element',
   templateUrl: './server-element.component.html',
   styleUrls: ['./server-element.component.css'],
   encapsulation: ViewEncapsulation.None // view is not encapsulated - CSS of this component is global , default option: Emulated - this component's CSS is valid only for its html, other is : Native  - ?
   })
```

## Local reference in template

```HTML
   <p>Add new Servers or blueprints!</p>
   <label>Server Name</label>
   <!-- <input type="text" class="form-control" [(ngModel)]="newServerName"> -->
   <input 
      type="text" 
      class="form-control"
      #serverNameInput>    <!-- local ref. can be used ONLY in template and holds the whole html element, but can be passed as argument and used then inside ts code-->
   <label>Server Content</label>
   <!-- <input type="text" class="form-control" [(ngModel)]="newServerContent"> -->
   <input 
      type="text"
      class="form-control"
      #serverContentInput>
   <br>
   <button
      class="btn btn-primary"
      (click)="onAddServer(serverNameInput, serverContentInput)">Add Server</button>
   <button
      class="btn btn-primary"
      (click)="onAddBlueprint(serverNameInput, serverContentInput)">Add Server Blueprint</button>
```

## Angular lifecycle
- https://www.udemy.com/course/the-complete-guide-to-angular-2/learn/lecture/6656102#bookmarks
- __ngOnChanges__ - called after a bound input property changes
   - reacts ONLY to changes of properties: if property holds an object it holds reference to object in memory - change of it doesn't change the reference,
     that is why when we change value on object, property that we want to check for changes needs to hold primitive, cause it changes with change of value of referenced object value
- __ngOnInit__    - called once the component is initialized
- __ngDoCheck__   - called during every change detection run, even when no change occured
- __ngAfterContentInit__   - called after content (ng-content) has been projected into view
- __ngAfterContentChecked__- called when projected content has been checked
- __ngAfterViewInit__      - called after the component's view has been inintialized
- __ngAfterViewChecked__   - called every time the view has been checked
- __ngOnDestroy__          - called once the component is about to be removed from DOM (e.g. with)

## SERVICES
Used for communication between components. Proper use by:
   -  including in __providers__ property od __@Component__ decorator
   -  injecting it as parameter of __constructor()__
   -  NOT instantiating "by hand" = serviceInstance = new Service();
   -  example:
   ```ts
      import { Component, EventEmitter, Input, Output } from '@angular/core';
      import { LoggingService } from '../logging.service';
      import { AccountsService } from '../accounts.service';

      @Component({
         selector: 'app-account',
         templateUrl: './account.component.html',
         styleUrls: ['./account.component.css'],
         providers: [LoggingService, AccountsService]
      })

      export class AccountComponent {
         @Input() account: {name: string, status: string};
         @Input() id: number;

         constructor(private loggingService: LoggingService, private accountsService: AccountsService) {}

         onSetTo(status: string) {
            this.accountsService.onStatusChanged(this.id, status);
            this.loggingService.logStatus(status);
         }
      }
   ```

   -  HIERARCHICAL INJECTOR - reach of injected service depends on injection level in __providers__ property of @Component:
      -  __AppModule__:           __The same instance__ of Service is available in __whole app__ (all components and all services)
         - or by decorating service with:
         ```ts
            @Injectable({
               providedIn: 'root'   // this way or by adding service in app.module.ts in providers we make service app-wide available
            })                      // Services can be loaded lazily by Angular (behind the scenes) and redundant code can be removed automatically. This can lead to a better performance and loading speed
         ```
      -  __AppComponent__:        __The same instance__ of Service is available for __all Components__ (but not for services)
      -  __Any other Component__: __The same instance__ of Service is available for __the Component and all its child components__ and overwrites the same service provided on higher level
      -  prevent above mentioned overwriting by removing service from __providers__ property, still need it in __constructor()__ as __private__ parameter

## ROUTING

   -  register routes in app.modules.ts:
   ```ts
      ...
      import { RouterModule, Routes } from '@angular/router';
      ...
      const appRoutes = [
         { path: '', component: HomeComponent },
         { path: 'users', component: UsersComponent },
         { path: 'servers', component: ServersComponent }
      ]

      @NgModule({ 
         ...
         imports: [
            BrowserModule,
            FormsModule,
            RouterModule.forRoot(appRoutes)
         ], 
   ```
   - nest routes: child routes need to have router-outlet in parent html
   ```TS
      const appRoutes = [
         { path: '', component: HomeComponent },
         { path: 'users', component: UsersComponent, children: [
            { path: ':id/:name', component: UserComponent }, // 'id' custom specified name, that can be retrieved inside component; ':' marks that this is dynamic part of path
         ] },
         { path: 'servers', component: ServersComponent, children: [
            { path: ':id', component: ServerComponent },
            { path: ':id/edit', component: EditServerComponent }
         ] }
      ]
   ```
   ```html
      <!-- servers.component.html -->
      <div class="row">
         <div class="col-xs-12 col-sm-4">
            <div class="list-group">
               <a
               [routerLink]="['/servers', server.id]"
               [queryParams]="{allowEdit: server.id === 3 ? 1 : 0}"
               fragment="loading"
               class="list-group-item"
               *ngFor="let server of servers">
               {{ server.name }}
               </a>
            </div>
         </div>
         <div class="col-xs-12 col-sm-4">
            <router-outlet></router-outlet>
            <!-- <button class="btn btn-primary" (click)="onReload()">Reload</button>
            <app-edit-server></app-edit-server>
            <hr>
            <app-server></app-server> -->
         </div>
      </div>
   ```

   - location for router with __router-outlet__ directive  in html - app.component.html:
   ```html
      <div class="row">
         <div class="col-xs-12 col-sm-10 col-md-8 col-sm-offset-1 col-md-offset-2">
            <router-outlet></router-outlet>
         </div>
      </div>
   ```
   - navigation with __routerLink__ directive - ensures no refresh of site: it's faster and keeps the state of app:
   ```html
      <ul class="nav nav-tabs">
        <li role="presentation" routerLinkActive="active" [routerLinkActiveOptions]="{ exact: true }"><a routerLink="/">Home</a></li>
        <li role="presentation" routerLinkActive="active"><a routerLink="/servers">Servers</a></li>
        <li role="presentation" routerLinkActive="active"><a [routerLink]="['/users']">Users</a></li>
      </ul>
   ```
      -  "/" ensures absolute path usage
      -  without "/" or "./" : relative path is used - string is appended to current url path
      -  "../../" : relative - go back 2 (or less or more) segments and then append string provided
      -  above start always from current path - path at which component.html including routerLink, that we define, is
   - __routerLinkActive__ - see code above: adds class to element with active routerLink
   - __[routerLinkActiveOptions]="{ exact: true }__ - ensures routerLinkActive works only on exact match, NOT when active link contains routerLink string
   - __this.router.navigate(['/']);__ - programmatic navigation with Router injected into component class
   - this.router.navigate(['server'], { relativeTo: this.route}); // with relativeTo property we can set a url=path, to which relative (without "/") path will be added
   - this.route.snapshot.params['id'] - retrieves value of 'id' param from current path (url) - if this is palced in ngOnInit function, then it doesnt get updated on url change
   - subscribe to route params to react to changes of params in url
      ```ts
         this.route.params.subscribe(
            (params: Params) => {
            this.user.id = params['id'];
            this.user.name = params['name'];
            }
         )
      ```
   - ### Setting URL
      - set path with routerLink property
      - set query with queryParams property (part of URL after "?", params separeted by "&")
      - set fragment with fragment property (last part of URL after "#")
      - in html
         ```html
            <a
               [routerLink]="['/servers', 1, 'edit']"
               [queryParams]="{allowEdit: '1', allowAnything: '1'}"
               fragment="loading"
               class="list-group-item"
               *ngFor="let server of servers">
               {{ server.name }}
            </a>
         ```
      - programmatically in TS with arguments of navigate method
         ```TS
            this.router.navigate(['/servers', id, 'edit'], { queryParams: {allowEdit: 1}, fragment: 'lalamido' });
         ```
      - navigate with keeping queryParams
         ```TS
            this.router.navigate(['edit'], {relativeTo: this.route, queryParamsHandling: 'preserve'});
         ```

   - ### Access control to routes ( https://github.com/kklapczynski/Udemy_Angular8_The-Complete-Guide__routing/tree/master/src/app )
      - It's done with Guards - services that can restrict access to routes; app-routing.module.ts : appRoutes:
      ```TS
         { path: 'servers', 
            // canActivate: [AuthGuard], // AuthGuard works for servers path and all its children
            canActivateChild: [AuthGuard],  // AuthGuard works for servers children paths only
            component: ServersComponent, 
            children: [
            { path: ':id', component: ServerComponent },
            { path: ':id/edit', component: EditServerComponent }
            ] 
         },
      ```
      - auth-guard.service.ts:
      ```TS
         @Injectable()
         export class AuthGuard implements CanActivate, CanActivateChild {
            
            constructor(private authService: AuthService, private router: Router) {}

            canActivate(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): Observable<boolean> | Promise<boolean> | boolean {
               return this.authService.isAuthenticated()
                     .then(
                        (authenticated: boolean) => {
                           if(authenticated) {
                                 return true;
                           } else {
                                 this.router.navigate(['/']);
                                 return false;
                           }
                        }
                     )
            }

            canActivateChild(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): Observable<boolean> | Promise<boolean> | boolean {
               return this.canActivate(route, state);
            }
         }
      ```
      - control navigating away from route with CanDeactivate interface:
         - app-routing.module.ts
         ```TS
         { path: ':id/edit', component: EditServerComponent, canDeactivate: [CanDeactivateGuard] } // CanDeactivateGuard implements CanDeactivate interface to decide if a route can be deactivated - navigated away from
         ```
         - can-deactivate-guard.service.ts:
         ```TS
            // to make this kind of guard reusable by any component, we need it to force component to have certain methods

            export interface CanComponentDeactivate {
               canDeactivate: () => Observable<boolean> | Promise<boolean> | boolean;
            }

            export class CanDeactivateGuard implements CanDeactivate<CanComponentDeactivate> {  
               // implementing CanDeactivate interface with component of type CanComponentDeactivate means this guard can be used only on components that implement CanComponentDeactivate interface
               // which ensures that this component has canDeactivate method returning boolean (straight away or as result of Observable or Promise)

               canDeactivate(                          // this canDeactivate() method will be called by the angular router once we try to leave the route
                  component: CanComponentDeactivate,  // component that guard is tied to in app-routing.module
                  currentRoute: ActivatedRouteSnapshot,
                  currentState: RouterStateSnapshot,
                  nextState?: RouterStateSnapshot) : Observable<boolean> | Promise<boolean> | boolean {   // "?" marks optional argument
               
                  return component.canDeactivate();   // with this angular router calls component's canDeactivate() method - if it returns true, navigation is allowed and follows
                                                      // this ties component's canDeactivate method with the router - CanDeactivateGuard service introduced in app-routing.module.ts uses canDeactivate() method defined in component
                                                      // mechanism of navigating is in Guard + Router, but logic for stoping navigation is in component
               }
            }
         ```
         - edit-server.component.ts
         ```TS
         export class EditServerComponent implements OnInit, CanComponentDeactivate {
            ...
            // other guards didn't need access to component's properties, but here it needs to check changesSaved and be provided as a service in app-routin.module
            canDeactivate(): Observable<boolean> | Promise<boolean> | boolean {
               if (!this.allowEdit) return true;
               if ((this.serverName !== this.server.name || this.serverStatus !== this.server.status) && !this.changesSaved) {
                  return confirm('Discard changes?');
               } else {
                  return true;
               }
               // console.log('edit-server.component: canDeactivate()');
               // return false;
            }
         ```
      - use static or dynamic (retrieved by resolver) data from router
         - app-routing.module.ts
         ```TS
         { path: 'servers', 
            // canActivate: [AuthGuard], // AuthGuard works for servers path and all its children
            canActivateChild: [AuthGuard],  // AuthGuard works for servers children paths only
            component: ServersComponent, 
            children: [
            { path: ':id', component: ServerComponent, resolve: {server: ServerResolver} }, // resolver works before component is loaded and stores in this case <Server> in data['server'] - the same data object as used for static data below
            { path: ':id/edit', component: EditServerComponent, canDeactivate: [CanDeactivateGuard] } // CanDeactivateGuard implements CanDeactivate interface to decide if a route can be deactivated - navigated away from
            ] 
         },
         // { path: 'not-found', component: PageNotFoundComponent },
         { path: 'not-found', component: ErrorComponent, data: {message: 'Error: page not found!!!'} },  // static data passed by router - to use in component; e.g. use 1 component with different paths and individual data
         ```
         - server-resolver.service.ts
         ```TS
         import { Resolve, ActivatedRouteSnapshot, RouterStateSnapshot } from "@angular/router";
         import { Observable } from "rxjs";
         import { ServersService } from "../servers.service";
         import { Injectable } from "@angular/core";

         interface Server {  // this interface should be in separate file
            id: number,
            name: string,
            status: string
         }

         @Injectable()   // cause we are injecting service into service
         export class ServerResolver implements Resolve<Server> {
            constructor(private serverService: ServersService){}
            resolve(
               route: ActivatedRouteSnapshot,
               state: RouterStateSnapshot
            ): Observable<Server> | Promise<Server> | Server {
               return this.serverService.getServer(parseInt(route.params['id']));  // here data required = server is instantly accesible, but reslove can be used to fetch something before loading the route
            }
         }
         ```
         - server.component.ts
         ```TS
         this.route.data.subscribe(
            (data: Data) => {
            this.server = data.server;
            }
         )
         ```

      - routing in old browsers or when server cannot set up index.html as a 404 page
         - app-routing.module.ts
         ```TS
         @NgModule({
            imports: [
               RouterModule.forRoot(appRoutes)
               // RouterModule.forRoot(appRoutes, {useHash: true}) // for angular route to work on real server we need to set its 404 page to index.html (so angular can handle everything)
            ],                                                      // if that is not possible or for old browsers 'useHash' setting adds '#' after root path to deal with that 
            exports: [RouterModule]
         })
         ```
   - ### NOTES to ROUTING
      - watch < a href="#" ...> in elements with attached [routerLink], cause it overwrites path setting with '#'

## OBSERVABLES
   - https://www.udemy.com/course/the-complete-guide-to-angular-2/learn/lecture/14466306#questions/8300194
   - use __rxjs__ : 
      - https://rxjs-dev.firebaseapp.com/
      - https://academind.com/learn/javascript/understanding-rxjs/
   - use Subject of rxjs insead of EventEmitter of Angular for intercomponent comunication
      - .next() replaces .emit()
      - NEED to UNSUBSCRIBE by hand opposite to EventEmitter, in which Angular handles unsubscription