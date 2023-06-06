# temp2

By default, Angular Material's tooltip does not support HTML content. However, you can achieve your desired functionality by creating a custom tooltip using Angular Material's overlay module and incorporating HTML content. Here's a step-by-step guide on how you can accomplish this:

1. Install Angular Material: If you haven't already, install Angular Material by running the following command in your project directory:

```
ng add @angular/material
```

2. Import required modules: In your module file (e.g., `app.module.ts`), import the necessary Angular Material modules:

```typescript
import { NgModule } from '@angular/core';
import { MatTooltipModule } from '@angular/material/tooltip';
import { OverlayModule } from '@angular/cdk/overlay';

@NgModule({
  imports: [
    // ...
    MatTooltipModule,
    OverlayModule
  ],
  // ...
})
export class AppModule { }
```

3. Create a custom tooltip directive: Create a new file, e.g., `html-tooltip.directive.ts`, and define a custom tooltip directive:

```typescript
import { Directive, ElementRef, HostListener, Input, OnDestroy, OnInit, Renderer2, TemplateRef, ViewContainerRef } from '@angular/core';
import { MatTooltip, MatTooltipDefaultOptions, MAT_TOOLTIP_DEFAULT_OPTIONS } from '@angular/material/tooltip';
import { Overlay, OverlayRef, PositionStrategy, ScrollStrategy } from '@angular/cdk/overlay';
import { TemplatePortal } from '@angular/cdk/portal';

@Directive({
  selector: '[htmlTooltip]'
})
export class HtmlTooltipDirective extends MatTooltip implements OnInit, OnDestroy {
  @Input('htmlTooltip') content!: string | TemplateRef<any>;

  private overlayRef: OverlayRef;

  constructor(
    overlay: Overlay,
    elementRef: ElementRef<HTMLElement>,
    private viewContainerRef: ViewContainerRef,
    private renderer: Renderer2,
    private defaultOptions: MatTooltipDefaultOptions
  ) {
    super(overlay, elementRef, defaultOptions);
  }

  ngOnInit(): void {
    super.ngOnInit();
    this.tooltipClass = 'html-tooltip';
    this.position = 'below';
  }

  ngOnDestroy(): void {
    super.ngOnDestroy();
    this.overlayRef.dispose();
  }

  @HostListener('mouseenter')
  show(): void {
    if (!this.overlayRef) {
      this.overlayRef = this.createOverlay();
      this.renderer.addClass(this.overlayRef.overlayElement, 'html-tooltip-overlay');
    }

    const position = this.overlay.position();
    position.withDefaultOffsetX(16).withDefaultOffsetY(16);
    position.withPositions([
      { originX: 'center', originY: 'bottom', overlayX: 'center', overlayY: 'top' },
      { originX: 'center', originY: 'top', overlayX: 'center', overlayY: 'bottom' },
      { originX: 'end', originY: 'center', overlayX: 'start', overlayY: 'center' },
      { originX: 'start', originY: 'center', overlayX: 'end', overlayY: 'center' }
    ]);

    const scrollStrategy: ScrollStrategy = this.overlay.scrollStrategies.reposition();
    const portal = new TemplatePortal(this.content, this.viewContainerRef);
    this.overlayRef.attach(portal);
    this.overlayRef.updatePositionStrategy(position);
    this.overlayRef.updateScrollStrategy(scrollStrategy);
  }

  private createOverlay(): OverlayRef {
    const overlayConfig = this.getOverlayConfig();
    return this.overlay.create(overlayConfig);
  }

  private getOverlayConfig(): any {
    const overlayConfig = this.defaultOptions.positionStrategy || this.overlay.position().global();
    overlayConfig.host = this.elementRef.nativeElement;
    overlayConfig.scrollStrategy
