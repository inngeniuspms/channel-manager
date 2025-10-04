# Innconnect Channel Manager API Documentation

### Introduction


This document describes how to interact with the Innconnect SOAP API. All API operations are `POST` requests to a single endpoint, using different `SOAPAction` headers and XML request bodies to specify the desired action.

The base URL for the API is: `https://app.inn-connect.com/api/innconnectapi`

**Note on XML Examples:** The `curl` examples in this guide use the `-d @filename.xml` syntax. This assumes the XML request body is saved in a file (e.g., `getrooms.xml`). You should save the provided XML examples into corresponding files or embed the XML content directly in your requests.

### Security Credentials

Each request must include your security credentials within the SOAP header. To become a partner and receive your `user` and `password` credentials, please contact `info@inngeniuspms.com`.
You will be asked for your IP addresses so it can be whitelisted.

The `propertyId` is the unique identifier for your property in the Innconnect system. You must be authorized to manage this property.

To find your `propertyId`:
1.  Log into your Innconnect account.
2.  Navigate to your property's page.
3.  Click on the "Property Details" button.
4.  The ID is displayed next to the "Property Information" tab label. For example, if you see "Property Information (ID: 192)", your `propertyId` is `192`.

---

### 1. Get Property Info (Rooms)

This operation retrieves all available room types and rate plans for a given property. The response provides the `InvTypeCode` (room type ID) and `RatePlanCode` (rate plan ID) needed for other API calls.

**`curl` Request:**

```bash
curl -X POST \
-H "Content-Type: text/xml; charset=UTF-8" \
-H 'SOAPAction: ""' \
-d @getrooms.xml \
https://app.inn-connect.com/api/innconnectapi
```

**Request Body (`getrooms.xml`):**

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:tns="http://innconnectapi.vega.com/">
    <soapenv:Header>
        <tns:Security xmlns="">
            <user>YOUR_USER_ID</user>
            <password>YOUR_PASSWORD</password>
            <propertyId>YOUR_PROPERTY_ID</propertyId>
        </tns:Security>
    </soapenv:Header>
    <soapenv:Body/>
</soapenv:Envelope>
```

**Successful Response:**

The response lists room types and rate plans. The `<key>` element combines the `InvTypeCode` and `RatePlanCode`, separated by a period (e.g., `184.185`).

```xml
<?xml version="1.0" encoding="UTF-8"?>
<S:Envelope xmlns:S="http://schemas.xmlsoap.org/soap/envelope/">
  <S:Body>
    <ns3:getPropertyInfoResponse xmlns:ns2="http://www.opentravel.org/OTA/2003/05" xmlns:ns3="http://innconnectapi.vega.com/">
      <propertyId>192</propertyId>
      <propertyName>Grand Test Hotel</propertyName>
      <roomTypes>
        <entry>
          <key>184.185</key>
          <value>Single Room/Standard Rate</value>
        </entry>
        <!-- ... other room type and rate plan entries ... -->
      </roomTypes>
    </ns3:getPropertyInfoResponse>
  </S:Body>
</S:Envelope>
```

---

### 2. Update Availability

This operation updates the number of available rooms (`BookingLimit`) for a specific room/rate combination over a date range.

**`curl` Request:**

```bash
curl -X POST \
-H "Content-Type: text/xml; charset=UTF-8" \
-H 'SOAPAction: "http://app.inn-connect.com/api/HotelAvailNotifRQ"' \
-d @availability.xml \
https://app.inn-connect.com/api/innconnectapi
```

**Request Body (`availability.xml`):**

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:tns="http://innconnectapi.vega.com/">
    <soapenv:Header>
        <tns:Security xmlns="">
            <user>YOUR_USER_ID</user>
            <password>YOUR_PASSWORD</password>
            <propertyId>YOUR_PROPERTY_ID</propertyId>
        </tns:Security>
    </soapenv:Header>
    <soapenv:Body>
        <OTA_HotelAvailNotifRQ Version="1.0" TimeStamp="2025-01-01T00:00:00" EchoToken="echo-abc123"
                               xmlns="http://www.opentravel.org/OTA/2003/05">
            <AvailStatusMessages>
                <AvailStatusMessage BookingLimit="55">
                    <StatusApplicationControl Start="2026-01-01" End="2026-01-30" InvTypeCode="184" RatePlanCode="185"/>
                </AvailStatusMessage>
            </AvailStatusMessages>
        </OTA_HotelAvailNotifRQ>
    </soapenv:Body>
</soapenv:Envelope>
```

**Successful Response:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<S:Envelope xmlns:S="http://schemas.xmlsoap.org/soap/envelope/">
  <S:Body>
    <ns2:OTA_HotelAvailNotifRS xmlns:ns2="http://www.opentravel.org/OTA/2003/05" xmlns:ns3="http://innconnectapi.vega.com/" EchoToken="echo-abc123">
      <ns2:Success/>
    </ns2:OTA_HotelAvailNotifRS>
  </S:Body>
</S:Envelope>
```

---

### 3. Update Minimum Length of Stay (MinLOS)

This operation sets the minimum length of stay for a specific room/rate combination over a date range.

**`curl` Request:**

```bash
curl -X POST \
-H "Content-Type: text/xml; charset=UTF-8" \
-H 'SOAPAction: "http://app.inn-connect.com/api/HotelAvailNotifRQ"' \
-d @minlos.xml \
https://app.inn-connect.com/api/innconnectapi
```

**Request Body (`minlos.xml`):**

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:tns="http://innconnectapi.vega.com/" >
    <soapenv:Header>
        <tns:Security xmlns="">
            <user>YOUR_USER_ID</user>
            <password>YOUR_PASSWORD</password>
            <propertyId>YOUR_PROPERTY_ID</propertyId>
        </tns:Security>
    </soapenv:Header>
    <soapenv:Body>
        <OTA_HotelAvailNotifRQ Version="1.0" TimeStamp="2025-01-01T00:00:00" EchoToken="echo-abc123" xmlns="http://www.opentravel.org/OTA/2003/05">
            <AvailStatusMessages>
                <AvailStatusMessage>
                    <StatusApplicationControl Start="2026-01-01" End="2026-01-30" InvTypeCode="184" RatePlanCode="185"/>
                    <LengthsOfStay>
                        <LengthOfStay Time="5" MinMaxMessageType="SetMinLOS"/>
                    </LengthsOfStay>
                </AvailStatusMessage>
            </AvailStatusMessages>
        </OTA_HotelAvailNotifRQ>
    </soapenv:Body>
</soapenv:Envelope>
```

**Successful Response:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<S:Envelope xmlns:S="http://schemas.xmlsoap.org/soap/envelope/">
  <S:Body>
    <ns2:OTA_HotelAvailNotifRS xmlns:ns2="http://www.opentravel.org/OTA/2003/05" xmlns:ns3="http://innconnectapi.vega.com/" EchoToken="echo-abc123">
      <ns2:Success/>
    </ns2:OTA_HotelAvailNotifRS>
  </S:Body>
</S:Envelope>
```

---

### 4. Update Rate Amount

This operation updates the price (`AmountBeforeTax`) for a specific room/rate combination over a date range.

**`curl` Request:**

```bash
curl -X POST \
-H "Content-Type: text/xml; charset=UTF-8" \
-H 'SOAPAction: "http://app.inn-connect.com/api/HotelRateAmountNotifRQ"' \
-d @rate.xml \
https://app.inn-connect.com/api/innconnectapi
```

**Request Body (`rate.xml`):**

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
                  xmlns:tns="http://innconnectapi.vega.com/">
    <soapenv:Header>
        <tns:Security xmlns="">
            <user>YOUR_USER_ID</user>
            <password>YOUR_PASSWORD</password>
            <propertyId>YOUR_PROPERTY_ID</propertyId>
        </tns:Security>
    </soapenv:Header>
    <soapenv:Body>
        <OTA_HotelRateAmountNotifRQ EchoToken="test" TimeStamp="2023-02-21T10:00:00" Version="1.0" xmlns="http://www.opentravel.org/OTA/2003/05">
            <RateAmountMessages>
                <RateAmountMessage>
                    <StatusApplicationControl InvTypeCode="184" RatePlanCode="185"/>
                    <Rates>
                        <Rate Start="2026-01-01" End="2026-03-05">
                            <BaseByGuestAmts>
                                <BaseByGuestAmt AmountBeforeTax="18.20"/>
                            </BaseByGuestAmts>
                        </Rate>
                    </Rates>
                </RateAmountMessage>
            </RateAmountMessages>
        </OTA_HotelRateAmountNotifRQ>
    </soapenv:Body>
</soapenv:Envelope>
```

**Successful Response:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<S:Envelope xmlns:S="http://schemas.xmlsoap.org/soap/envelope/">
  <S:Body>
    <ns2:OTA_HotelRateAmountNotifRS xmlns:ns2="http://www.opentravel.org/OTA/2003/05" xmlns:ns3="http://innconnectapi.vega.com/" EchoToken="test">
      <ns2:Success/>
    </ns2:OTA_HotelRateAmountNotifRS>
  </S:Body>
</S:Envelope>
```

---

### 5. Retrieving Reservations

This operation retrieves new, modified, and cancelled reservations for the property. The request fetches all reservations that have changed since the last successful poll.
Once you have processed the reservations, you should send a NotifReportRQ to acknowledge receipt otherwise the same reservations will be returned on the next poll.

You can use the booking engine for creating test reservations.
And use https://app.inn-connect.com/cmtest.jsp to update them (manually change status to Modify or Cancel).


See the Confirmation documentation for details.

**`curl` Request:**

```bash
curl -X POST \
-H "Content-Type: text/xml; charset=UTF-8" \
-H 'SOAPAction: "http://app.inn-connect.com/api/ReadRQ"' \
-d @readrq.xml \
https://app.inn-connect.com/api/innconnectapi
```

**Request Body (`readrq.xml`):**

The request body uses an `OTA_ReadRQ` message. The `HotelReadRequest` element can be left empty to retrieve all pending reservation updates.

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
                  xmlns:tns="http://innconnectapi.vega.com/">
    <soapenv:Header>
        <tns:Security xmlns="">
            <user>YOUR_USER_ID</user>
            <password>YOUR_PASSWORD</password>
            <propertyId>YOUR_PROPERTY_ID</propertyId>
        </tns:Security>
    </soapenv:Header>
    <soapenv:Body>
        <OTA_ReadRQ xmlns="http://www.opentravel.org/OTA/2003/05">
            <ReadRequests>
                <HotelReadRequest>
                </HotelReadRequest>
            </ReadRequests>
        </OTA_ReadRQ>
    </soapenv:Body>
</soapenv:Envelope>
```

**Successful Response:**

A successful response will contain an `OTA_HotelResNotifRQ` element, which includes a list of reservations within `HotelReservations`. Each reservation includes guest details, stay dates, rate information, and a unique ID.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<S:Envelope xmlns:S="http://schemas.xmlsoap.org/soap/envelope/">
    <S:Body>
        <ns2:OTA_ResRetrieveRS xmlns:ns2="http://www.opentravel.org/OTA/2003/05"
                               xmlns:ns3="http://innconnectapi.vega.com/" Version="1.0"
                               TimeStamp="2025-09-14T08:29:27.639Z">
            <ns2:Success/>
            <ns2:ReservationsList>
                <!--There can be multiple reservations in this list-->
                <ns2:HotelReservation CreateDateTime="2024-10-10T23:18:41.823Z" ResStatus="Book">
                    <!--ResStatus can be Book|Modify|Cancel-->
                    <ns2:POS>
                        <ns2:Source>
                            <ns2:BookingChannel Primary="true" Type="7">
                                <!--The origin of the booking-->
                                <ns2:CompanyName Code="inn-connect.com"/>
                            </ns2:BookingChannel>
                        </ns2:Source>
                    </ns2:POS>
                    <!--Use ID below for sending the confirmation-->
                    <ns2:UniqueID Type="14" ID="2881183"/>
                    <ns2:RoomStays>
                        <ns2:RoomStay>
                            <ns2:RoomTypes>
                                <ns2:RoomType RoomType="Double Room" RoomTypeCode="485414"/>
                            </ns2:RoomTypes>
                            <ns2:RatePlans>
                                <ns2:RatePlan RatePlanCode="1422538" EffectiveDate="2024-11-12" ExpireDate="2024-11-14"
                                              RatePlanName="Summer Non-refundable">
                                </ns2:RatePlan>
                            </ns2:RatePlans>
                            <ns2:RoomRates>
                                <ns2:RoomRate RatePlanType="Summer Non-refundable" RatePlanCode="1422538"
                                              EffectiveDate="2024-11-12">
                                    <ns2:Rates>
                                        <ns2:Rate>
                                            <ns2:Total AmountBeforeTax="9.9" AmountAfterTax="9.9" CurrencyCode="USD"/>
                                        </ns2:Rate>
                                    </ns2:Rates>
                                </ns2:RoomRate>
                            </ns2:RoomRates>
                            <ns2:GuestCounts>
                                <ns2:GuestCount AgeQualifyingCode="10" Count="2"/>
                                <ns2:GuestCount AgeQualifyingCode="8" Count="0"/>
                            </ns2:GuestCounts>
                            <ns2:TimeSpan End="2024-11-13" Start="2024-11-12"/>
                            <ns2:Total AmountBeforeTax="9.9" AmountAfterTax="9.9" CurrencyCode="USD"/>
                            <ns2:BasicPropertyInfo/>
                            <ns2:ResGuestRPHs>
                                <ns2:ResGuestRPH RPH="1"/>
                            </ns2:ResGuestRPHs>
                        </ns2:RoomStay>
                    </ns2:RoomStays>
                    <ns2:ResGuests>
                        <ns2:ResGuest ResGuestRPH="1">
                            <ns2:Profiles>
                                <ns2:ProfileInfo>
                                    <ns2:Profile ProfileType="1">
                                        <ns2:Customer>
                                            <ns2:PersonName>
                                                <ns2:GivenName>John</ns2:GivenName>
                                                <ns2:Surname>Doe</ns2:Surname>
                                            </ns2:PersonName>
                                            <ns2:Address/>
                                        </ns2:Customer>
                                    </ns2:Profile>
                                </ns2:ProfileInfo>
                            </ns2:Profiles>
                        </ns2:ResGuest>
                    </ns2:ResGuests>
                    <ns2:ResGlobalInfo>
                        <ns2:Guarantee>
                            <ns2:GuaranteesAccepted>
                                <ns2:GuaranteeAccepted>
                                    <ns2:PaymentCard CardCode="VI" CardType="1" CardNumber="4111111111111111"
                                                     ExpireDate="2812" SeriesCode="111">
                                        <ns2:CardHolderName>John Doe</ns2:CardHolderName>
                                    </ns2:PaymentCard>
                                </ns2:GuaranteeAccepted>
                            </ns2:GuaranteesAccepted>
                        </ns2:Guarantee>
                        <ns2:Total AmountBeforeTax="9.9" AmountAfterTax="9.9" CurrencyCode="USD"/>
                        <ns2:HotelReservationIDs>
                            <ns2:HotelReservationID ResID_Type="14" ResID_Value="2024-10-10 23:18 041"/>
                        </ns2:HotelReservationIDs>
                        <ns2:Profiles>
                            <ns2:ProfileInfo>
                                <ns2:Profile ProfileType="1">
                                    <ns2:Customer>
                                        <ns2:PersonName>
                                            <ns2:GivenName>Johnny</ns2:GivenName>
                                            <ns2:Surname>Doe</ns2:Surname>
                                        </ns2:PersonName>
                                        <ns2:Telephone PhoneNumber="7137750917"/>
                                        <ns2:Email>johnmdoe@adomainofcustomer.com</ns2:Email>
                                        <ns2:Address>
                                            <ns2:AddressLine>999222 Country Place Rd.</ns2:AddressLine>
                                            <ns2:CityName>City</ns2:CityName>
                                            <ns2:PostalCode>77355</ns2:PostalCode>
                                            <ns2:StateProv>TX</ns2:StateProv>
                                            <ns2:CountryName>US</ns2:CountryName>
                                        </ns2:Address>
                                    </ns2:Customer>
                                </ns2:Profile>
                            </ns2:ProfileInfo>
                        </ns2:Profiles>
                        <ns2:Comments>
                            <ns2:Comment Name="PaymentMethod">
                                <ns2:Text>PaymentMethod:cc</ns2:Text>
                            </ns2:Comment>
                        </ns2:Comments>
                        <ns2:GuestCounts/>
                    </ns2:ResGlobalInfo>
                </ns2:HotelReservation>
            </ns2:ReservationsList>
        </ns2:OTA_ResRetrieveRS>
    </S:Body>
</S:Envelope>

```

---


### 6. Confirmation of Reservations
After processing the reservations received from the `OTA_ResRetrieveRS` response, you must send a `NotifReportRQ` to acknowledge receipt. This prevents the same reservations from being returned in subsequent polls.

**`curl` Request:**

```bash
curl -X POST \
-H "Content-Type: text/xml; charset=UTF-8" \
-H 'SOAPAction: "http://app.inn-connect.com/api/NotifReportRQ"' \
-d @confirm.xml \
https://app.inn-connect.com/api/innconnectapi
```

**Request Body (`confirm.xml`):**

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
                  xmlns:tns="http://innconnectapi.vega.com/" >
    <soapenv:Header>
        <tns:Security xmlns="">
            <user>YOUR_USER_ID</user>
            <password>YOUR_PASSWORD</password>
            <propertyId>YOUR_PROPERTY_ID</propertyId>
        </tns:Security>
    </soapenv:Header>
    <soapenv:Body>
        <OTA_NotifReportRQ EchoToken="NotifReport_1" TimeStamp="2024-01-01T12:00:00" Version="1.0"
                           xmlns="http://www.opentravel.org/OTA/2003/05">
            <Success/>
            <NotifDetails>
                <HotelNotifReport>
                    <HotelReservations>
                        <HotelReservation>
                            <UniqueID Type="14" ID="YOUR_BOOKING_ID"/>
                        </HotelReservation>
                    </HotelReservations>
                </HotelNotifReport>
            </NotifDetails>
        </OTA_NotifReportRQ>
    </soapenv:Body>
</soapenv:Envelope>


```

**Successful Response:**
```xml
<?xml version="1.0" encoding="UTF-8"?><S:Envelope xmlns:S="http://schemas.xmlsoap.org/soap/envelope/">
    <S:Body>
        <ns2:OTA_NotifReportRS xmlns:ns2="http://www.opentravel.org/OTA/2003/05" xmlns:ns3="http://innconnectapi.vega.com/">
            <ns2:Success/>
        </ns2:OTA_NotifReportRS>
    </S:Body>
</S:Envelope>
```




### 7. Error Handling

If a request fails due to invalid data, incorrect credentials, or other issues, the API will return a SOAP response containing an `<Errors>` element.

**Error Response Example (`error_response.xml`):**

This example shows an error for a user not authorized to access the specified property.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<S:Envelope xmlns:S="http://schemas.xmlsoap.org/soap/envelope/">
  <S:Body>
    <ns2:OTA_HotelRateAmountNotifRS xmlns:ns2="http://www.opentravel.org/OTA/2003/05" xmlns:ns3="http://innconnectapi.vega.com/">
      <ns2:Errors>
        <ns2:Error Type="1">Not allowed to access this property. 192</ns2:Error>
      </ns2:Errors>
    </ns2:OTA_HotelRateAmountNotifRS>
  </S:Body>
</S:Envelope>
```


### 8. Best Practices and Recommendations

To ensure a stable and reliable integration, please follow these recommendations.

**General Integration:**
*   **Initial Testing:** Before writing application code, use a tool like `curl` to manually send requests for each API operation. This is the fastest way to verify your XML structure, headers, and authentication details.
*   **Error Handling:** Implement logic to parse the `<Errors>` element in API responses. Your application should be able to handle different error types gracefully, log the error details, and implement retry mechanisms where appropriate.
*   **Logging:** Log all API requests and responses

**Data Synchronization:**
*   **Daily Full Data Push:** To prevent discrepancies, it is critical to send a full year of availability, rates, and restrictions (like MinLOS) at least once per day. This ensures that the channel manager's data is a perfect mirror of your system's data.
*   **Use `EchoToken` for Idempotency:** Where available, use the `EchoToken` attribute in your requests. This will help to associate requests with responses and can prevent duplicate processing in case of retries.

**Reservation Management:**
*   **Frequent Polling:** Poll for new, modified, and cancelled reservations every 5-10 minutes. This ensures your property management system receives updates in a timely manner, reducing the risk of overbookings or missed cancellations.
*   **Acknowledge Reservations Promptly:** Immediately after successfully processing reservations from a `OTA_ResRetrieveRS` response, send the `NotifReportRQ` confirmation. Failure to do so will cause the API to send the same reservations again in the next poll.


### 9. Real-Time Reservation Notifications (Push Trigger)

To receive reservation updates in near real-time instead of waiting for the next scheduled poll, you can provide a "Push Trigger" endpoint. When a new reservation is created, modified, or cancelled, Innconnect will send a `GET` request to your specified URL. This request serves as a signal for your system to immediately poll for the latest reservation data.

**How It Works:**

1.  **Provide an Endpoint:** You provide a secure HTTPS endpoint from your PMS that can accept `GET` requests. This URL can include the placeholder `{hotelid}`, which Innconnect will dynamically replace with the relevant property ID.
2.  **Configuration:** Contact us to have this endpoint configured for your API user account.
3.  **Trigger:** When a reservation event occurs for one of your properties, Innconnect will immediately send an empty `GET` request to your configured endpoint.
    *   **Example Endpoint:** `https://pms.example.com/innconnect/trigger?hotelid={hotelid}`
    *   **Actual Request Sent:** `GET https://pms.example.com/innconnect/trigger?hotelid=192`
4.  **Action:** Upon receiving this `GET` request, your system should:
    *   Immediately respond with an `HTTP 200 OK` status to acknowledge receipt.
    *   Asynchronously initiate a "Retrieving Reservations" request (`OTA_ReadRQ`, as described in Section 5) to fetch the full details of the new/updated reservation(s).

This mechanism allows your system to retrieve reservation data within seconds of the event, significantly reducing latency compared to periodic polling.
