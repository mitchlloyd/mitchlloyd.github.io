---
title: Perspective on Names
permalink: perspective-on-names
---

Some guidelines for naming software constructs are a matter of taste and vary
with technologies. But whatever the context, names should be chosen from the
perspective of the caller. _Code Complete_ explains this concept stating that a
good routine name "speaks to the problem rather than the solution". Naming
methods and functions from the caller's perspective may seem like a stylistic
choice at first but the practice improves the quality of abstractions in a code
base.

## Hide Implementation Details

An obvious way to use the wrong perspective when naming is to reveal the inner
workings of some object or function. Consider the following example where EMF
stands for Embedded Metrics Format, an AWS logging protocol used to send metrics
to the AWS CloudWatch service.

```go
func (s *Server) ReportUser() {
    // ...
    if err != nil {
        // Implementation details leaked!
        s.emfLogger.Log("ReportFailed", 1, "Count")
    }
}
```

It's easy to make this mistake after you've spent a couple hours thinking about
nothing but the EMF protocol. It's a mental shift to remember that the caller
shouldn't be tied to the underlying technologies used for metrics.

```go
func (s *Server) ReportUser() {
    // ...
    if err != nil {
        // Object named in the domain of Server.ReportUser
        s.metricSender.Send("ReportFailed", 1, "Count")
    }
}
```

The fact that `Server` sends metrics using the EMF protocol _is_ important in
the context of configuring the `Server` struct. Perhaps here a developer may
choose to use the CloudWatch HTTP API or an EMF logger. But notice that as the
object moves from the configuration context into the `Server` context, its name
changes.

```go
func main() {
    s := Server{metricSender: NewEMFLogger()}
}
```

## Describe What Things Do

A more subtle mistake is to choose names based on the input being passed rather
than the output created or the side effect produced. In the next example, the
name `kinesisEventProcessor` provides no new information at the call site. The
caller already knew that this was a Kinesis record.

```go
func handler(ctx context.Context, req events.KinesisEvent) error {
    for _, record := range req.Records {
        kinesisRecordProcessor.Process(record)
    }
}
```

Renaming this object is an opportunity to explain the domain to the reader.

```go
func handler(ctx context.Context, req events.KinesisEvent) error {
    for _, record := range req.Records {
        highScorePublisher.Publish(record)
    }
}
```

## Care With Named Parameters

Sometimes parameters are named and organized for convenience in a function's
implementation rather than the caller's. In this example, parameters that happen
to be shared between three functions are grouped under `SharedParams` to avoid
superficial duplication in type definitions.

```go
// Functions used at different call sites.
NewHighScore(SharedParams{UserID: "user-1", Game: "Chess"}, 1_000)
NewGameWin(SharedParams{UserID: "user-2", Game: "Checkers"})
NewPurchase(
    SharedParams{UserID: "user-3", Game: "Zelda"}, 
    Payment{Amount: 100, Currency: "USD"}
)

```

But to the callers it's not useful to know that these function implementations
share parameters. Instead, define named parameters for clarity at the call
sites.

```go
NewHighScore(HighScoreParams{UserID: "user-1", Game: "Chess", Score: 1_000})
NewGameWin(GameWinParams{UserID: "user-2", Game: "Checkers"})
NewPurchase(PurchaseParams{
    UserID: "user-3",
    Game: "Zelda",
    Payment{Amount: 100, Currency: "USD"},
}) 
```

## Redundant Words

It can seem helpful to create functions tailored to each caller's use case. In
this example, `imageCropper` provides a purpose-built `CropProfileImage` method
for `SaveProfile`.

```go
func (s *Server) SaveProfile(id string, image []byte) {
    cropped := s.imageCropper.CropProfileImage(image)
    // ...
}
```

Use the repetition of the word "Profile" seen in the context of the caller
(`SaveProfile`) and the method being called (`CropProfileImage`) as a signal
that `imageCropper` has a poor boundary between its responsibility and its
caller's. In this case the word "Profile" in `CropProfileImage` is a way for
`imageCropper` to identify its caller is to customize cropping behavior:

```go
func (c *ImageCropper) CropProfileImage(image []byte) []byte {
    return c.crop(Dimensions{Height: 300, Width: 300})
}

func (c *ImageCropper) CropAlbumArt(image []byte) []byte {
    return c.crop(Dimensions{Height: 1200, Width: 1200})
}
```

The caller shouldn't have to identify itself by calling a specific method or
with a flag argument like `type: PROFILE`. Instead name the method concisely in
the context of the caller and parameterize the information needed.

```go
func (s *Server) SaveProfile(id string, image []byte) { cropped :=
    s.imageCropper.Crop(Dimensions{Height: 300, Width: 300})
    // ...
}
```

When I'm reviewing code I've written, I like to do a top-down (or outside-in)
reading to look to make sure I'm using object, method, and function names that
work from the perspective of the callers.
