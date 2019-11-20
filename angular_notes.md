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