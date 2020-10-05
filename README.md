# Richardson Maturity Model

> According to Martin Fowler

This model breaks down the principal elements of a REST approach into three steps.

These introduce resources, http verbs, and hypermedia controls.

## Level 0: The Swamp of POX

Using HTTP as a transport system for remote interactions without using any of the mechanisms of the web. (RPC)

Requesting available appointments will be modelled in the following way.

Request

`POST /appointmentService HTTP/1.1 [headers]`

```yaml
openSlotRequest:
  date: “2020-01-04”
  doctor: ”genci”
```

Response

`HTTP/1.1 200 OK [headers]`

```yaml
openSlotList:
  - slot:
      start: 1400
      end: 1450
      doctor:
        id: ”genci”
  - slot:
      start: 1600
      end: 1650
      doctor:
        id: ”genci”
```

Booking an appointment will be the next request.

Request

`POST /appointmentService HTTP/1.1`

```yaml
appointmentRequest:
  slot:
    doctor: ”genci”
    start: 1400
    end: 1650
  patient:
    id: ”johncena”
```

Response

`HTTP/1.1 200 OK [headers]`

```yaml
appointment:
  slot:
    doctor: "genci"
    start: 1600
    end: 1650
  patient:
    id:" johncena"
```

Here we see that the same endpoint is accessed for making two different kinds of requests: one for showing available slots, and the other for booking an appointment. To someone who has used upper levels of the maturity model, this looks like an obvious flaw.

If the appointment is booked, there will be an error response message.

`HTTP/1.1 200 OK [headers]`

```yaml
appointmentRequestFailure:
  slot:
    doctor: "genci"
    start: 1600
    end: 1650
  patient:
    id: "johncena"
  reason: "Slot not available"
```

This is another contradiction of this level, where we got a success status code and an error message.

## Level 1: Resources

The first step towards the maturity level is to introduce resources. Rather than making all requests to a single endpoint, we will start requesting individual resources.

Request

`POST /doctors/genci HTTP/1.1 [headers]`

```yaml
openSlotRequest:
  date: "2020-01-04"
```

Response

```yaml
openSlotList:
  - slot
      id: 1
      start: 1400
      end: 1450
      doctor: "genci"
  - slot
      id: 2
      start: 1600
      end: 1650
      doctor: "genci"
```

Each slot resource can be accessed individually.

Booking an appointment for an individual slot.

Request

`POST /slots/1 HTTP/1.1 [headers]`

```yaml
appointmentRequest:
  patient:
    id: "johncena"
```

Response

`HTTP/1.1 200 OK`

```yaml
appointment:
  slot:
    id: 1
    doctor: "genci"
    start: 1400
    end: 1450
  patient:
    id: "johncena"
```

## Level 2: HTTP Verbs

We've still used only POST verbs on levels 0 and 1. HTTP level 2 introduces HTTP verbs specific to their usage in the request.

Request

`GET /doctors/genci/slots?date=20200104&status=open HTTP/1.1 [headers]`

```yaml
openSlotList:
  - slot:
      id: 1
      start: 1400
      end: 1450
      doctor: "genci"
  - slot:
      id: 2
      start: 1600
      end: 1650
      doctor: "genci"
```

GET is a safe operation, that doesn't make any significant changes to the state of anything.

It also supports caching.

Booking an appointment.

Request

`POST /slots/1 HTTP/1.1 [headers]`

```yaml
appointmentRequest:
  patient:
    id: "johncena"
```

Response

```bash
HTTP/1.1 201 Created
Location: slots/1/appointment
[headers]
```

```yaml
appointment:
  slot:
    id: 1
    doctor: "genci"
    start: 1400
    end: 1450
  patient:
    id: "johncena"
```

The 201 status indicates that there is a new resource and the location attribute represents and URI that the client can use to access this resource in the future.

If something goes wrong.

```bash
HTTP/1.1 409 Conflict [headers]
```

```yaml
openSlotList:
  - slot:
      id: 2
      start: 1600
      end: 1650
      doctor: "genci"
```

The 409 status indicates that something went trong. Rather than a 200 status with an error response, in level 2 we explicitly use error responses like this one.

## Level 3: Hypermedia Controls

This final levels introduces HATEOAS: Hypermedia As The Engine Of Application State. Helluva name.

Request

`GET /doctors/genci/slots?date=20100104&status=open HTTP/1.1 [headers]`

Response (wait for it)

`HTTP/1.1 200 OK [headers]`

```yaml
openSlotList:
  - slot:
      id: 1
      doctor: "genci"
      start: 1600
      end: 16:50
      _links:
        book:
          uri: "/slots/1"
  - slot:
      id: 2
      doctor: "genci"
      start: 1700
      end: 17:50
      _links:
        book:
          uri: "/slots/2"
```

Interestingly, a slot element now includes a link object which contains an uri that shows us where to book an appointment.

Following this uri:

`POST /slots/1 HTTP/1.1 [headers]`

```yaml
appointmentRequest:
  patient:
    id: "johncena"
```

We will get this kind of response:

```bash
HTTP/1.1 201 Created
Location: /slots/1/appointment
[headers]
```

```yaml
appointment:
  slot:
    id: 1
    doctor: "genci"
    start: 1600
    end: 1650
  patient:
    id: "johncena"
  _links:
    cancel:
      uri: "/slots/1/appointment"
    changeTime:
      uri: "/doctors/genci/slots?date=20200104&status=open"
    self:
      uri: "/slots/1/appointment"
```

The links give the developer a hint to what is possible to do next on this resource. This level adds discoverability and self-documentation.

**Level 3 is a pre-condition of REST**, according to Roy Fielding.

And, you can stop calling any HTTP-based interface that you built a REST API.
