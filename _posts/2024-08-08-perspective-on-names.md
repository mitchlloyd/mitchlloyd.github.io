---
title: Perspective on Names
permalink: perspective-on-names
---

Some guidelines for naming software constructs are a matter of taste and vary
with technologies. But whatever the context, names should be chosen from the
perspective of the caller. [_Code Complete_](https://amzn.to/4cZ3EDj) explains
this concept stating that a good routine name "speaks to the problem rather than
the solution". Naming methods and functions from the caller's perspective may
seem like a stylistic choice at first but the practice improves the quality of
abstractions in a codebase.

## Hide implementation details of objects and methods

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

In this case, the arguments for sending metrics with EMF logging or with
CloudWatch HTTP APIs should be the same. It's easy to make this naming mistake
after you've spent a couple hours thinking about nothing but the EMF protocol.
It's a mental shift to remember that the caller shouldn't be tied to the
underlying technologies used for metrics.

```go
func (s *Server) ReportUser() {
    // ...
    if err != nil {
        // Object named in the context of Server.ReportUser
        s.metricSender.Send("ReportFailed", 1, "Count")
    }
}
```

The fact that `Server` sends metrics using the EMF protocol _is_ important in
the context of configuring the `Server` struct. Perhaps here a developer may
choose between using the CloudWatch HTTP API or an EMF logger. But notice that
as the object moves from the configuration context into the `Server` context,
its name changes.

```go
func main() {
    s := Server{metricSender: NewEMFLogger()}
}
```

## But describe what things do

Another common mistake is choosing names based on the input being passed rather
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

## Hide implementation details with parameters

Sometimes parameters are named and organized for convenience in a function's
implementation rather than the caller's. One situation where this can happen is
when a method routes parameters to several collaborators.

```go
func (s *Server) PostComment(user *User, device *Device, text string) {
    s.Authorize(AuthorizeParams{
        phoneServiceParams: {
            phoneNumber: user.PhoneNumber,
        },
        identityServiceParams: {
            username: user.Username
        },
        reputationServiceParams: {
            fingerprint: device.Fingerprint,
        },
    })

    // ...
}
```

Rather than revealing how these parameters will be split between internal
collaborators, in this minimal example we can provide more focused parameters.

```go
func (s *Server) PostComment(user *User, device *Device, text string) {
    s.Authorize(AuthorizeParams{
        phoneNumber: user.PhoneNumber,
        username: user.Username
        fingerprint: device.Fingerprint,
    })

    // ...
}
```

If you run into a case where a caller has to know about the underlying services
being used, reevaluate whether the method is a useful abstraction or whether the
caller should interact with these services directly.

## Notice overlapping context

Focusing on the ergonomics of the caller doesn't mean we should try to eliminate
all of its responsibilities. Instead we want boundaries that support deep rather
than of shallow modules (see [_A Philosophy of Software Design_ - Modules Should
be deep](https://amzn.to/3LjYWUQ)). Looking for names that bleed into the
caller's domain can help spot problem areas.

In this example, `imageCropper` provides a purpose-built `CropProfileImage`
method for `SaveProfile`.

```go
func (s *Server) SaveProfile(id string, image []byte) {
    cropped := s.imageCropper.CropProfileImage(image)
    // ...
}
```

The repetition of the word "Profile" seen in the context of the caller
(`SaveProfile`) and the method being called (`CropProfileImage`) is a signal
that `imageCropper` has a poor boundary between its responsibility and its
caller's. In this case the word "Profile" in `CropProfileImage` is a way for
`imageCropper` to identify its caller to customize cropping behavior:

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

## Optimize for clarity at the call site

When I'm reviewing code I've written, I like to do a top-down (or outside-in)
reading to make sure I'm using object, method, and function names that work from
the perspective of the callers. 

Ideally the names educate a reader about the domain from a specific vantage
point without introducing lower-level details. Developing a sense for naming can
help quickly identify places to reconsider an approach.
