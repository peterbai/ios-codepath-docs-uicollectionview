<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [Overview](#overview)
- [Collection view setup](#collection-view-setup)
  - [Step 1: Determine what data to display in the collection view](#step-1-determine-what-data-to-display-in-the-collection-view)
  - [Step 2: Add collection view to view controller](#step-2-add-collection-view-to-view-controller)
  - [Step 3: Create custom class for prototype cell](#step-3-create-custom-class-for-prototype-cell)
  - [Step 4: Configure prototype cell](#step-4-configure-prototype-cell)
  - [Step 5: Set prototype cell's reuse identifier](#step-5-set-prototype-cells-reuse-identifier)
  - [Step 6: Set the collection view's data source](#step-6-set-the-collection-views-data-source)
- [Collection view usage](#collection-view-usage)
  - [Respond to cell selection](#respond-to-cell-selection)
    - [Segue to another view controller](#segue-to-another-view-controller)
    - [Handling cell selection progammatically](#handling-cell-selection-progammatically)
- [Further reading](#further-reading)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Overview
[Collection views][uicollectionview] are used to display *cells* in customizable layouts. Here are some examples:

![Sample collection views](Images/Examples/uicollectionview-samples.jpg)

[uicollectionview]:https://developer.apple.com/library/ios/documentation/UIKit/Reference/UICollectionView_class/
*Left to right: Marvin, Flickr, Storehouse*

This guide is a quick intro to setting up and using a simple collection view with Interface Builder.

## Collection view setup

We'll be creating a grid of colors with labels showing their RGB values.

![Example project screenshot](Images/Docs/collectionView-overview.jpg)

### Step 1: Determine what data to display in the collection view

Decide what kind of data you'd like to show. For simple text, you might use an array of strings. In our example, we'll use a function that takes an index path and returns a corresponding UIColor. Type this code into your view controller's class definition:

```swift
let totalColors: Int = 100
func colorForIndexPath(indexPath: NSIndexPath) -> UIColor {
    if indexPath.row >= totalColors {
        return UIColor.blackColor()	// return black if we get an unexpected row index
    }
    
    var hueValue: CGFloat = CGFloat(indexPath.row) / CGFloat(totalColors)
    return UIColor(hue: hueValue, saturation: 1.0, brightness: 1.0, alpha: 1.0)
}
```

### Step 2: Add collection view to view controller
In your Storyboard, drag a `Collection View` (not `Collection View Controller`) from the Object Library into your view controller. Use the Assistant Editor to add an outlet for the collection view in your view controller.

![Adding collection view to view controller](Images/Docs/Gif/01-adding-collectionView.gif)

### Step 3: Create custom class for prototype cell

The `Collection View` comes with a `Collection View Cell` prototype. Create a new class to use as a template for these cells.

Select `File -> New -> File... -> iOS -> Source -> Cocoa Touch Class`
and create a new subclass of `UICollectionViewCell`. Name it `ColorCell`. Back in your Storyboard, select your prototype cell, and in the Identity Inspector, set its custom class property to the new class you just created.

![Creating custom cell class](Images/Docs/CustomCell-class.jpg)

### Step 4: Configure prototype cell

Change the background color of the Collection View to white. Add a `Label` from the Object Library to the prototype cell, and change its placeholder text to `Color`. Use the Assistant Editor to add an outlet for the label in the `ColorCell` class. Name it `colorLabel`.

![Configuring custom cell](Images/Docs/Gif/02-configuring-ColorCell.gif)

Your ColorCell class should have the following code:

```swift
class ColorCell: UICollectionViewCell {
    @IBOutlet weak var colorLabel: UILabel!
}
```

### Step 5: Set prototype cell's reuse identifier

Select the cell in the Storyboard and give it a unique identifier in the Attributes Inspector. This allows the Collection View to reuse instances of our prototype cell.

![Setting reuse identifier](Images/Docs/CustomCell-identifier.jpg)

### Step 6: Set the collection view's data source

In your view controller's `viewDidLoad` method, set the collection view's data source:

```swift
class ViewController: UIViewController {
    @IBOutlet weak var collectionView: UICollectionView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        collectionView.dataSource = self
    }
```

Add an extension that implements the `UICollectionViewDataSource` protocol. Implement the `collectionView(_:numberOfItemsInSection:)` and `collectionView(_:cellForItemAtIndexPath:)` methods.

```swift
extension ViewController: UICollectionViewDataSource {
    func collectionView(collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
        return totalColors
    }
    
    func collectionView(collectionView: UICollectionView, cellForItemAtIndexPath indexPath: NSIndexPath) -> UICollectionViewCell {
        let cell = collectionView.dequeueReusableCellWithReuseIdentifier("com.codepath.ColorCell", forIndexPath: indexPath) as! ColorCell
        let cellColor = colorForIndexPath(indexPath)
        cell.backgroundColor = cellColor
        
        if CGColorGetNumberOfComponents(cellColor.CGColor) == 4 {
            let redComponent = CGColorGetComponents(cellColor.CGColor)[0] * 255
            let greenComponent = CGColorGetComponents(cellColor.CGColor)[1] * 255
            let blueComponent = CGColorGetComponents(cellColor.CGColor)[2] * 255
            cell.colorLabel.text = String(format: "%.0f, %.0f, %.0f", redComponent, greenComponent, blueComponent)
        }
        
        return cell
    }
}
```

Build and run the demo app. Your collection view should show 100 colored cells, each with a different hue and a label of its RGB value.

## Collection view usage

### Respond to cell selection
A common event you'll need to respond to is the user selecting a cell in the collection view.

#### Segue to another view controller
Often, you'll want a cell selection to push a new view controller with details about that cell. In your Storyboard, control-drag from the prototype cell onto a new view controller, and select the appropriate segue under the `Selection Segue` section.

![Selection segue](Images/Docs/Gif/03-cell-selection-segue.gif)

#### Handling cell selection progammatically
You can also respond programmatically by implementing the `UICollectionViewDelegate` [`collectionView(_:didSelectItemAtIndexPath:)`][didSelectItemAtIndexPath] method.

In your view controller's `viewDidLoad` method, add a line to set the collection view's delegate.

```swift
collectionView.delegate = self
```

Then, add an extension to your view controller that adopts the UICollectionViewDelegate protocol, and implement the above method. The following code prints a cell's row index when it is selected.

```swift
extension ViewController: UICollectionViewDelegate {
    func collectionView(collectionView: UICollectionView, didSelectItemAtIndexPath indexPath: NSIndexPath) {
        println("Selected cell number: \(indexPath.row)")
    }
}
```

[didSelectItemAtIndexPath]: https://developer.apple.com/library/ios/documentation/UIKit/Reference/UICollectionViewDelegate_protocol/index.html#//apple_ref/occ/intfm/UICollectionViewDelegate/collectionView:didSelectItemAtIndexPath:


## Further reading
* [Apple's collection view programming guide](https://developer.apple.com/library/ios/documentation/WindowsViews/Conceptual/CollectionViewPGforIOS/Introduction/Introduction.html)