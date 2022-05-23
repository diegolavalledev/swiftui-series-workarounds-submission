# Loopy Carousel

Submission to the SwiftUI Series **Workarounds** challenge.

![Loopy Carousel](screenshot.gif)

# Workaround details

In order to create the illusion of looping through the cards/images infinitely, we create two additional copies of the original deck and we tape them toghether making a 3x-length strip.

When the user taps on a card we reposition the strip using a calculated offset so that we are always set around the middle of the tape. We do this explicitely without animation. The goal is for the user not to notice any visual changes at all during this phase.

Finally we transition in the direction of the target card by moving the offset one more time, this time using `withAnimation()`. The animation will create the carousel effect by sliding while scaling the card at the center.

# Relevant snippets

All the relevant code is in the [ContentView](LoopyCarousel/LoopyCarousel/ContentView.swift) source.

The layout consists of an outer, fixed `HStack` which wraps an inner, shifting HStack containing all the cards.

```swift
HStack(spacing: 0) {
  HStack(spacing: Self.spacing) {
      ForEach(0 ..< loopCount, id: \.self) { i in
      Card(content: cardLabels[i % cardLabels.count], isSelected: .constant(current == i))
        .onTapGesture { onSelect(i) }
    }
  }
  .offset(x: offset)
}
.clipped()
```

Some calculated properties help us render the deck and determine the current offset.

```swift
private var loopCount: Int {
  cardLabels.count * 3
}

private var cardWidth: CGFloat {
  Card.normalWidth + Self.spacing
}

private var evenShift: CGFloat {
  loopCount.isMultiple(of: 2)
  ? -cardWidth / 2
  : 0
}

private var offset: CGFloat {
  cardWidth * CGFloat(loopCount / 2 - current) + evenShift
}
```

Our handler for the tap action does the work of preparing the view before triggering the animated transition.

```swift
private func onSelect(_ index: Int) -> () {
  let jump = abs(current - index)
  if (index >= cardLabels.count * 2) {
    current = cardLabels.count + (index % cardLabels.count) - jump
    withAnimation { current += jump }
  } else if (index < cardLabels.count) {
    current = cardLabels.count + (index % cardLabels.count) + jump
    withAnimation { current -= jump }
  } else {
    withAnimation { current = index }
  }
}
```
