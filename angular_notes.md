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

- __*ngIf__  ( * marks structural directive - changes structure of DOM: adds or removes elements )

   ```html
    <p *ngIf="this.isServerCreated">Server >>> {{this.serverName}} <<< was created.</p>
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
- __*ngFor__ - loops through an array to add multiple elements

   ```html
    <app-server *ngFor="let server of servers"></app-server>
    <div *ngFor="let logItem of log; let i = index">{{ logItem }}</div>
   ```
- 