![bubblebee](http://idigitalcitizen.files.wordpress.com/2009/07/1920x1200-bumblebee88.jpg)


Bubblebee is an abstract text processing and pattern matching engine in Swift for iOS and OSX. This provides support for things like markdown and basic HTML tags to be properly converted from raw text to expected style using NSAttributedString. Example markdown engine is included. 

## Features

- Abstract and simple. Creating patterns is simple, yet flexible.
- Fast. Only one pass is make through the raw string to minimize parse time.
- Simple concise codebase at just a few hundred LOC.

## Examples

This is a simple example, but showcases a powerful use case.

```swift
//first we create our label to show our text
let label = UILabel(frame: CGRectMake(0, 65, view.frame.size.width, 400))
label.numberOfLines = 0
view.addSubview(label)

//we create this textAttachment to show our embedded image
var textAttachment = NSTextAttachment(data: nil, ofType: nil)

//the raw text we have. 
let rawText = "Hello I am *red* and I am _bold_. Here is an image: ![](http://vluxe.io/assets/images/logo.png)"

//create our BumbleBee object.
let bee = BumbleBee()

//our red text pattern
bee.add("*?*", recursive: false) { (pattern: String, text: String, start: Int) -> (String, [NSObject : AnyObject]?) in
    let replace = pattern[advance(pattern.startIndex, 1)...advance(pattern.endIndex, -2)]
    return (replace,[NSForegroundColorAttributeName: UIColor.redColor()])
}
//the bold pattern
bee.add("_?_", recursive: false) { (pattern: String, text: String, start: Int) -> (String, [NSObject : AnyObject]?) in
    let replace = pattern[advance(pattern.startIndex, 1)...advance(pattern.endIndex, -2)]
    return (replace,[NSFontAttributeName: UIFont.boldSystemFontOfSize(17)])
}
//the image pattern
bee.add("![?](?)", recursive: false, matched: { (pattern: String, text: String, start: Int) in
    let range = pattern.rangeOfString("]")
    if let end = range {
        let findRange = pattern.rangeOfString("(")
        if let startRange = findRange {
            let url = pattern[advance(startRange.startIndex, 1)..<advance(pattern.endIndex, -1)]
			//using Skeets(), we can easily fetch the remote image
            ImageManager.sharedManager.fetch(url, progress: { (Double) in
                }, success: { (data: NSData) in
                    let img = UIImage(data: data)
                    textAttachment.image = img
                    textAttachment.bounds = CGRect(x: 0, y: 0, width: img.size.width, height: img.size.height)
                    label.setNeedsDisplay() //tell our label to redraw now that we have our image
                    
                }, failure: { (error: NSError) in
            })
        }
        return (bee.attachmentString,[NSAttachmentAttributeName: textAttachment]) // embed an attachment
    }
    return ("",nil) //don't change anything, not a match
})
//now that we have our patterns, we call process and get the NSAttributedString
let attrString = bee.process(rawText)
label.attributedText = attrString
```

Which looks like:

![example](https://raw.githubusercontent.com/daltoniam/bumblebee/assets/example.png)

Image Loading Library:
[Skeets](https://github.com/daltoniam/Skeets)

## Requirements

Bubblebee requires at least iOS 7/OSX 10.10 or above.

## Installation

Add the `bubblebee.xcodeproj` to your Xcode project. Once that is complete, in your "Build Phases" add the `bubblebee.framework` to your "Link Binary with Libraries" phase.

## TODOs

- [ ] Complete Docs
- [ ] Add Unit Tests
- [ ] Add [Rouge](https://github.com/acmacalister/Rouge) Installation Docs

## License

Bubblebee is licensed under the Apache v2 License.

## Contact

### Dalton Cherry
* https://github.com/daltoniam
* http://twitter.com/daltoniam
* http://daltoniam.com