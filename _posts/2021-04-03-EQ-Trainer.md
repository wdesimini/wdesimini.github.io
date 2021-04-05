---
title: "EQ Trainer"
date: 2021-04-03
tags: [AVFoundation, Macaw, Swift]
---

EQ Trainer plays frequency bands using either Sine waves or Pink Noise to train your mixing ear and recognize specific frequencies in a mix. Users are able to track their skills through statistics as well as configure settings to more effectively practice and improve.

Libraries used:
- [AVFoundation](https://developer.apple.com/documentation/avfoundation) for Sine wave generator
- [Macaw](https://github.com/exyte/Macaw) for animated statistics graph
- [UIKit](https://developer.apple.com/documentation/uikit) for UI/controls

[Link to EQ Trainer in the App Store](https://apps.apple.com/us/app/eq-trainer/id1472810579)

Code sample - Reusable `MacawView` subclass `WillyBarGraphView` to create/update bar graphs -

```swift
import Foundation

protocol AxisValue {
    var axisText: String { get }
}

protocol XAxisValue: AxisValue, Comparable, Hashable {}

protocol YAxisValue: AxisValue, BinaryInteger {}
```

```swift
import Macaw

class WillyBarGraphView<XValueType: XAxisValue, YValueType: YAxisValue>: MacawView {
    var yValuesByXValues: [XValueType: YValueType] {
        didSet { layoutSubviews() }
    }
    
    private let inset: Double
    private let axisAlpha: Double
    private let crossSectionAlpha: Double
    private let axisTextPadding: Double = 16
    private let yAxisStep: YValueType
    private let axisColor: Color
    private let barColor: Color
    private let selectedBarColor: Color
    
    init(
        yValuesByXValues: [XValueType: YValueType],
        inset: Double = 64,
        axisAlpha: Double = 0.25,
        crossSectionAlpha: Double = 0.10,
        axesColor: Color = .white,
        barColor: Color = .black,
        selectedBarColor: Color = .blue,
        yAxisStep: YValueType
    ) {
        self.yValuesByXValues = yValuesByXValues
        self.inset = inset
        self.axisAlpha = axisAlpha
        self.crossSectionAlpha = crossSectionAlpha
        self.yAxisStep = yAxisStep
        self.axisColor = axesColor
        self.barColor = barColor
        self.selectedBarColor = selectedBarColor
        super.init(frame: .zero)
    }
    
    @objc required convenience init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    override func layoutSubviews() {
        super.layoutSubviews()
        
        node = Group(contents: xAxisNodes() + yAxisNodes() + barNodes(), place: .identity)
    }
    
    private var xValues: [XValueType] {
        yValuesByXValues.keys.sorted()
    }
    
    private var height: Double {
        Double(frame.size.height) - inset
    }
    
    private var width: Double {
        Double(frame.size.width) - inset
    }
    
    private var maxValue: YValueType {
        yValuesByXValues.values.sorted().last ?? 0
    }
    
    private func xAxisNodes() -> [Node] {
        var nodes = [Node]()
        
        let width = self.width
        let axisLine = Line(x1: 0, y1: width, x2: width, y2: width)
        nodes.append(axisLine.stroke(fill: axisColor.with(a: axisAlpha)))
        
        let xValues = self.xValues
        let increment = width / Double(xValues.count)
        let height = self.height
        
        for i in 1...xValues.count {
            let text = Text(
                text: xValues[i - 1].axisText,
                align: .max,
                baseline: .mid,
                place: .move(dx: Double(i) * increment, dy: height + axisTextPadding)
            )
            
            text.fill = axisColor
            
            nodes.append(text)
        }
        
        return nodes
    }
    
    private var yAxisStepCount: Int {
        let unroundedAxisValueCount = Double(maxValue) / Double(yAxisStep)
        let roundedAxisValueCount = Int(unroundedAxisValueCount)
        return Double(roundedAxisValueCount) == unroundedAxisValueCount ? roundedAxisValueCount : roundedAxisValueCount + 1
    }
    
    private func yAxisNodes() -> [Node] {
        var nodes = [Node]()
        
        let height = self.height
        let axisLine = Line(x1: 0, y1: 0, x2: 0, y2: height)
        nodes.append(axisLine.stroke(fill: axisColor.with(a: axisAlpha)))
        
        let stepCount = yAxisStepCount
        let increment = height / Double(stepCount)
        let width = self.width
        
        for i in 1...stepCount {
            let y = height - (Double(i) * increment)
            let crossSectionLine = Line(x1: 0, y1: y, x2: width, y2: y)
            nodes.append(crossSectionLine.stroke(fill: axisColor.with(a: crossSectionAlpha)))
            
            let text = Text(
                text: (YValueType(i) * yAxisStep).axisText,
                align: .max,
                baseline: .mid,
                place: .move(dx: -axisTextPadding, dy: y)
            )
            
            text.fill = axisColor
            
            nodes.append(text)
        }
        
        return nodes
    }
    
    private func barHeightsByXValue() -> [XValueType: Double] {
        let height = self.height
        let max = Double(maxValue)
        var heightsByXValue = [XValueType: Double]()
        
        for (x, y) in yValuesByXValues {
            heightsByXValue[x] = height * Double(y) / max
        }
        
        return heightsByXValue
    }
    
    private func barNodes() -> [Node] {
        let xValues = self.xValues
        let barHeightsByXValue = self.barHeightsByXValue()
        let barWidth = width / Double(xValues.count)
        let height = self.height
        var nodes = [Node]()
        
        for i in 0...xValues.count - 1 {
            let xValue = xValues[i]
            let barHeight = barHeightsByXValue[xValue]!
            let rect = Rect(x: Double(i) * barWidth, y: height - barHeight, w: barWidth, h: barHeight)
            nodes.append(rect.fill(with: barColor))
        }
        
        return nodes
    }
}
```
