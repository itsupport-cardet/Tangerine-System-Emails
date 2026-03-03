# Tangerine – System Emails Reference

Full catalogue of every automated email sent by the platform: trigger, recipients, subject, and HTML body preview.

---

## Table of Contents

| # | Email | App | Trigger |
|---|-------|-----|---------|
| 1 | [New Booking – Vendor Notification](#1-new-booking--vendor-notification) | `customers` | New booking created |
| 2 | [Booking Confirmed by Vendor](#2-booking-confirmed-by-vendor) | `customers` | Status → confirmed, vendor_confirm = True |
| 3a | [Booking Confirmed by Admin → Customer](#3a-booking-confirmed-by-admin--customer) | `customers` | Status → confirmed, vendor_confirm = False |
| 3b | [Booking Confirmed by Admin → Vendor](#3b-booking-confirmed-by-admin--vendor) | `customers` | Status → confirmed, vendor_confirm = False |
| 4 | [Booking Cancelled by Customer → Vendor](#4-booking-cancelled-by-customer--vendor) | `customers` | Status → cancelled, customer_cancel = True |
| 5 | [Booking Cancelled by Vendor → Customer](#5-booking-cancelled-by-vendor--customer) | `customers` | Status → cancelled, vendor_cancel = True |
| 6a | [Booking Cancelled by Admin → Customer](#6a-booking-cancelled-by-admin--customer) | `customers` | Status → cancelled, no cancel flag |
| 6b | [Booking Cancelled by Admin → Vendor](#6b-booking-cancelled-by-admin--vendor) | `customers` | Status → cancelled, no cancel flag |
| 7 | [Booking Status Updated](#7-booking-status-updated) | `customers` | Any other status change |
| 8 | [Booking Auto-Expired](#8-booking-auto-expired) | `customers` | Management command `--notify-customers` |
| 9 | [Vendor Welcome Email](#9-vendor-welcome-email) | `vendors` | New vendor account created |
| 10 | [Newsletter Broadcast](#10-newsletter-broadcast) | `newsletter` | NewsletterSend created with send_now = True |

---

## 1. New Booking – Vendor Notification

| Field | Value |
|-------|-------|
| **File** | `customers/signals.py` → `send_sms_new_booking()` |
| **Trigger** | `post_save` on `ActivityBooking`, `created=True` |
| **To** | Vendor ⚠️ currently hardcoded to `bokisangelov@gmail.com` — should be vendor email |
| **From** | ⚠️ hardcoded `bokis.angelov@innovade.eu` — should be `settings.DEFAULT_FROM_EMAIL` |
| **Subject** | `New booking` |

**Body:**

```html
<!DOCTYPE html>
<html>
<head>
  <style>
    body { font-family: Arial, sans-serif; color: #333; max-width: 600px; margin: 0 auto; padding: 20px; }
    .header { background: linear-gradient(135deg, #ff6b35, #f7931e); color: white; padding: 24px; border-radius: 10px 10px 0 0; text-align: center; }
    .content { background: #f9f9f9; padding: 24px; border-radius: 0 0 10px 10px; }
    .detail-box { background: white; border: 2px solid #ff6b35; border-radius: 8px; padding: 16px; margin: 16px 0; }
    .btn { display: inline-block; padding: 12px 24px; border-radius: 5px; text-decoration: none; font-weight: bold; margin: 8px 4px; }
    .btn-accept { background: #28a745; color: white; }
    .btn-reject { background: #dc3545; color: white; }
    .deadline { background: #fff3cd; border-left: 4px solid #ffc107; padding: 12px; margin: 16px 0; }
    .footer { text-align: center; font-size: 13px; color: #888; margin-top: 24px; }
  </style>
</head>
<body>
  <div class="header">
    <h1>🍊 New Booking Request</h1>
    <p>Action required – please respond before the deadline</p>
  </div>
  <div class="content">
    <p>Hi <strong>{vendor.business_name}</strong>,</p>
    <p>You have received a new booking request. Please review the details below and accept or reject.</p>

    <div class="detail-box">
      <h3>📋 Booking Details</h3>
      <p><strong>Activity:</strong> {activity_name}</p>
      <p><strong>Date:</strong> {booking.date}</p>
      <p><strong>Quantity:</strong> {booking.quantity} people</p>
    </div>

    <div class="detail-box">
      <h3>👤 Customer Details</h3>
      <p><strong>Name:</strong> {customer.first_name} {customer.last_name}</p>
      <p><strong>Phone:</strong> {customer.phone}</p>
    </div>

    <div class="deadline">
      ⏰ <strong>Response Deadline:</strong> {deadline_str}<br>
      If you do not respond, the booking will be auto-expired.
    </div>

    <div style="text-align:center; margin-top: 24px;">
      <a href="{accept_url}" class="btn btn-accept">✅ Accept Booking</a>
      <a href="{reject_url}" class="btn btn-reject">❌ Reject Booking</a>
    </div>
  </div>
  <div class="footer">
    <p>© Tangerine Platform. This is an automated message. Please do not reply.</p>
  </div>
</body>
</html>
```

---

## 2. Booking Confirmed by Vendor

| Field | Value |
|-------|-------|
| **File** | `customers/signals.py` → `send_sms_booking_status()` |
| **Trigger** | `post_save` on `ActivityBooking`, status = `confirmed`, `vendor_confirm = True` |
| **To** | Customer |
| **From** | ⚠️ hardcoded `bokis.angelov@innovade.eu` — should be `settings.DEFAULT_FROM_EMAIL` |
| **Subject** | `Booking Confirmed by Vendor` |

**Body:**

```html
<!DOCTYPE html>
<html>
<head>
  <style>
    body { font-family: Arial, sans-serif; color: #333; max-width: 600px; margin: 0 auto; padding: 20px; }
    .header { background: linear-gradient(135deg, #28a745, #20c997); color: white; padding: 24px; border-radius: 10px 10px 0 0; text-align: center; }
    .content { background: #f9f9f9; padding: 24px; border-radius: 0 0 10px 10px; }
    .detail-box { background: white; border: 2px solid #28a745; border-radius: 8px; padding: 16px; margin: 16px 0; }
    .footer { text-align: center; font-size: 13px; color: #888; margin-top: 24px; }
  </style>
</head>
<body>
  <div class="header">
    <h1>✅ Booking Confirmed!</h1>
    <p>Great news – your booking has been confirmed by the vendor</p>
  </div>
  <div class="content">
    <p>Hi <strong>{customer.first_name}</strong>,</p>
    <p>Great news! Your booking has been confirmed by the vendor.</p>

    <div class="detail-box">
      <h3>📋 Booking Details</h3>
      <p><strong>Booking ID:</strong> #{booking.id}</p>
      <p><strong>Activity:</strong> {activity_name}</p>
      <p><strong>Date:</strong> {booking.date}</p>
      <p><strong>Quantity:</strong> {booking.quantity} people</p>
    </div>

    <p>📞 If you have any questions, you can contact the vendor directly at <strong>{vendor_phone}</strong>.</p>
    <p>We look forward to seeing you there!</p>
    <p>Best regards,<br><strong>The Tangerine Team</strong></p>
  </div>
  <div class="footer">
    <p>© Tangerine Platform. This is an automated message. Please do not reply.</p>
  </div>
</body>
</html>
```

---

## 3a. Booking Confirmed by Admin → Customer

| Field | Value |
|-------|-------|
| **File** | `customers/signals.py` → `send_sms_booking_status()` |
| **Trigger** | `post_save` on `ActivityBooking`, status = `confirmed`, `vendor_confirm = False` |
| **To** | Customer |
| **From** | ⚠️ hardcoded `bokis.angelov@innovade.eu` — should be `settings.DEFAULT_FROM_EMAIL` |
| **Subject** | `Booking Confirmed by Administrator` |

**Body:**

```html
<!DOCTYPE html>
<html>
<head>
  <style>
    body { font-family: Arial, sans-serif; color: #333; max-width: 600px; margin: 0 auto; padding: 20px; }
    .header { background: linear-gradient(135deg, #007bff, #6610f2); color: white; padding: 24px; border-radius: 10px 10px 0 0; text-align: center; }
    .content { background: #f9f9f9; padding: 24px; border-radius: 0 0 10px 10px; }
    .detail-box { background: white; border: 2px solid #007bff; border-radius: 8px; padding: 16px; margin: 16px 0; }
    .footer { text-align: center; font-size: 13px; color: #888; margin-top: 24px; }
  </style>
</head>
<body>
  <div class="header">
    <h1>✅ Booking Confirmed by Administrator</h1>
  </div>
  <div class="content">
    <p>Hi <strong>{customer.first_name}</strong>,</p>
    <p>Your booking has been confirmed by the Tangerine system administrator.</p>

    <div class="detail-box">
      <h3>📋 Booking Details</h3>
      <p><strong>Booking ID:</strong> #{booking.id}</p>
      <p><strong>Activity:</strong> {activity_name}</p>
      <p><strong>Date:</strong> {booking.date}</p>
      <p><strong>Quantity:</strong> {booking.quantity} people</p>
    </div>

    <p>📞 If you have any questions, you can contact the vendor directly at <strong>{vendor_phone}</strong>.</p>
    <p>Best regards,<br><strong>The Tangerine Team</strong></p>
  </div>
  <div class="footer">
    <p>© Tangerine Platform. This is an automated message. Please do not reply.</p>
  </div>
</body>
</html>
```

---

## 3b. Booking Confirmed by Admin → Vendor

| Field | Value |
|-------|-------|
| **File** | `customers/signals.py` → `send_sms_booking_status()` |
| **Trigger** | Same as 3a |
| **To** | Vendor |
| **From** | ⚠️ hardcoded `bokis.angelov@innovade.eu` — should be `settings.DEFAULT_FROM_EMAIL` |
| **Subject** | `Booking Confirmed by Administrator` |

**Body:**

```html
<!DOCTYPE html>
<html>
<head>
  <style>
    body { font-family: Arial, sans-serif; color: #333; max-width: 600px; margin: 0 auto; padding: 20px; }
    .header { background: linear-gradient(135deg, #007bff, #6610f2); color: white; padding: 24px; border-radius: 10px 10px 0 0; text-align: center; }
    .content { background: #f9f9f9; padding: 24px; border-radius: 0 0 10px 10px; }
    .detail-box { background: white; border: 2px solid #007bff; border-radius: 8px; padding: 16px; margin: 16px 0; }
    .footer { text-align: center; font-size: 13px; color: #888; margin-top: 24px; }
  </style>
</head>
<body>
  <div class="header">
    <h1>📋 Booking Confirmed by Administrator</h1>
  </div>
  <div class="content">
    <p>Hi <strong>{vendor.business_name}</strong>,</p>
    <p>A booking for your activity has been confirmed by the Tangerine system administrator.</p>

    <div class="detail-box">
      <h3>📋 Booking Details</h3>
      <p><strong>Booking ID:</strong> #{booking.id}</p>
      <p><strong>Activity:</strong> {activity_name}</p>
      <p><strong>Date:</strong> {booking.date}</p>
      <p><strong>Quantity:</strong> {booking.quantity} people</p>
    </div>

    <div class="detail-box">
      <h3>👤 Customer Details</h3>
      <p><strong>Name:</strong> {customer.first_name} {customer.last_name}</p>
      <p><strong>Phone:</strong> {customer.phone}</p>
    </div>

    <p>Best regards,<br><strong>The Tangerine Team</strong></p>
  </div>
  <div class="footer">
    <p>© Tangerine Platform. This is an automated message. Please do not reply.</p>
  </div>
</body>
</html>
```

---

## 4. Booking Cancelled by Customer → Vendor

| Field | Value |
|-------|-------|
| **File** | `customers/signals.py` → `send_sms_booking_status()` |
| **Trigger** | `post_save` on `ActivityBooking`, status = `cancelled`, `customer_cancel = True` |
| **To** | Vendor |
| **From** | ⚠️ hardcoded `bokis.angelov@innovade.eu` — should be `settings.DEFAULT_FROM_EMAIL` |
| **Subject** | `Booking Cancelled by Customer` |

**Body:**

```html
<!DOCTYPE html>
<html>
<head>
  <style>
    body { font-family: Arial, sans-serif; color: #333; max-width: 600px; margin: 0 auto; padding: 20px; }
    .header { background: linear-gradient(135deg, #dc3545, #c82333); color: white; padding: 24px; border-radius: 10px 10px 0 0; text-align: center; }
    .content { background: #f9f9f9; padding: 24px; border-radius: 0 0 10px 10px; }
    .detail-box { background: white; border: 2px solid #dc3545; border-radius: 8px; padding: 16px; margin: 16px 0; }
    .footer { text-align: center; font-size: 13px; color: #888; margin-top: 24px; }
  </style>
</head>
<body>
  <div class="header">
    <h1>❌ Booking Cancelled by Customer</h1>
  </div>
  <div class="content">
    <p>Hi <strong>{vendor.business_name}</strong>,</p>
    <p>A customer has cancelled their booking for one of your activities.</p>

    <div class="detail-box">
      <h3>📋 Booking Details</h3>
      <p><strong>Booking ID:</strong> #{booking.id}</p>
      <p><strong>Activity:</strong> {activity_name}</p>
      <p><strong>Date:</strong> {booking.date}</p>
      <p><strong>Quantity:</strong> {booking.quantity} people</p>
    </div>

    <div class="detail-box">
      <h3>👤 Customer Details</h3>
      <p><strong>Name:</strong> {customer.first_name} {customer.last_name}</p>
      <p><strong>Phone:</strong> {customer.phone}</p>
    </div>

    <p>The slot is now available again for other bookings.</p>
    <p>Best regards,<br><strong>The Tangerine Team</strong></p>
  </div>
  <div class="footer">
    <p>© Tangerine Platform. This is an automated message. Please do not reply.</p>
  </div>
</body>
</html>
```

---

## 5. Booking Cancelled by Vendor → Customer

| Field | Value |
|-------|-------|
| **File** | `customers/signals.py` → `send_sms_booking_status()` |
| **Trigger** | `post_save` on `ActivityBooking`, status = `cancelled`, `vendor_cancel = True` |
| **To** | Customer |
| **From** | ⚠️ hardcoded `bokis.angelov@innovade.eu` — should be `settings.DEFAULT_FROM_EMAIL` |
| **Subject** | `Booking Cancelled by Vendor` |

**Body:**

```html
<!DOCTYPE html>
<html>
<head>
  <style>
    body { font-family: Arial, sans-serif; color: #333; max-width: 600px; margin: 0 auto; padding: 20px; }
    .header { background: linear-gradient(135deg, #dc3545, #c82333); color: white; padding: 24px; border-radius: 10px 10px 0 0; text-align: center; }
    .content { background: #f9f9f9; padding: 24px; border-radius: 0 0 10px 10px; }
    .detail-box { background: white; border: 2px solid #dc3545; border-radius: 8px; padding: 16px; margin: 16px 0; }
    .footer { text-align: center; font-size: 13px; color: #888; margin-top: 24px; }
  </style>
</head>
<body>
  <div class="header">
    <h1>❌ Booking Cancelled by Vendor</h1>
  </div>
  <div class="content">
    <p>Hi <strong>{customer.first_name}</strong>,</p>
    <p>We're sorry to let you know that the vendor has cancelled your booking.</p>

    <div class="detail-box">
      <h3>📋 Booking Details</h3>
      <p><strong>Booking ID:</strong> #{booking.id}</p>
      <p><strong>Activity:</strong> {activity_name}</p>
      <p><strong>Date:</strong> {booking.date}</p>
      <p><strong>Quantity:</strong> {booking.quantity} people</p>
    </div>

    <p>📞 For more information or to reschedule, please contact the vendor at <strong>{vendor_phone}</strong>.</p>
    <p>We apologise for the inconvenience. You can search for alternative activities on the platform.</p>
    <p>Best regards,<br><strong>The Tangerine Team</strong></p>
  </div>
  <div class="footer">
    <p>© Tangerine Platform. This is an automated message. Please do not reply.</p>
  </div>
</body>
</html>
```

---

## 6a. Booking Cancelled by Admin → Customer

| Field | Value |
|-------|-------|
| **File** | `customers/signals.py` → `send_sms_booking_status()` |
| **Trigger** | `post_save` on `ActivityBooking`, status = `cancelled`, `vendor_cancel = False` AND `customer_cancel = False` |
| **To** | Customer |
| **From** | ⚠️ hardcoded `bokis.angelov@innovade.eu` — should be `settings.DEFAULT_FROM_EMAIL` |
| **Subject** | `Booking Cancelled by Administrator` |

**Body:**

```html
<!DOCTYPE html>
<html>
<head>
  <style>
    body { font-family: Arial, sans-serif; color: #333; max-width: 600px; margin: 0 auto; padding: 20px; }
    .header { background: linear-gradient(135deg, #6c757d, #495057); color: white; padding: 24px; border-radius: 10px 10px 0 0; text-align: center; }
    .content { background: #f9f9f9; padding: 24px; border-radius: 0 0 10px 10px; }
    .detail-box { background: white; border: 2px solid #6c757d; border-radius: 8px; padding: 16px; margin: 16px 0; }
    .footer { text-align: center; font-size: 13px; color: #888; margin-top: 24px; }
  </style>
</head>
<body>
  <div class="header">
    <h1>❌ Booking Cancelled by Administrator</h1>
  </div>
  <div class="content">
    <p>Hi <strong>{customer.first_name}</strong>,</p>
    <p>Your booking has been cancelled by the Tangerine system administrator.</p>

    <div class="detail-box">
      <h3>📋 Booking Details</h3>
      <p><strong>Booking ID:</strong> #{booking.id}</p>
      <p><strong>Activity:</strong> {activity_name}</p>
      <p><strong>Date:</strong> {booking.date}</p>
      <p><strong>Quantity:</strong> {booking.quantity} people</p>
    </div>

    <p>📞 If you have any questions, please contact support or reach the vendor at <strong>{vendor_phone}</strong>.</p>
    <p>We apologise for the inconvenience.</p>
    <p>Best regards,<br><strong>The Tangerine Team</strong></p>
  </div>
  <div class="footer">
    <p>© Tangerine Platform. This is an automated message. Please do not reply.</p>
  </div>
</body>
</html>
```

---

## 6b. Booking Cancelled by Admin → Vendor

| Field | Value |
|-------|-------|
| **File** | `customers/signals.py` → `send_sms_booking_status()` |
| **Trigger** | Same as 6a |
| **To** | Vendor |
| **From** | ⚠️ hardcoded `bokis.angelov@innovade.eu` — should be `settings.DEFAULT_FROM_EMAIL` |
| **Subject** | `Booking Cancelled by Administrator` |

**Body:**

```html
<!DOCTYPE html>
<html>
<head>
  <style>
    body { font-family: Arial, sans-serif; color: #333; max-width: 600px; margin: 0 auto; padding: 20px; }
    .header { background: linear-gradient(135deg, #6c757d, #495057); color: white; padding: 24px; border-radius: 10px 10px 0 0; text-align: center; }
    .content { background: #f9f9f9; padding: 24px; border-radius: 0 0 10px 10px; }
    .detail-box { background: white; border: 2px solid #6c757d; border-radius: 8px; padding: 16px; margin: 16px 0; }
    .footer { text-align: center; font-size: 13px; color: #888; margin-top: 24px; }
  </style>
</head>
<body>
  <div class="header">
    <h1>❌ Booking Cancelled by Administrator</h1>
  </div>
  <div class="content">
    <p>Hi <strong>{vendor.business_name}</strong>,</p>
    <p>A booking for your activity has been cancelled by the Tangerine system administrator.</p>

    <div class="detail-box">
      <h3>📋 Booking Details</h3>
      <p><strong>Booking ID:</strong> #{booking.id}</p>
      <p><strong>Activity:</strong> {activity_name}</p>
      <p><strong>Date:</strong> {booking.date}</p>
      <p><strong>Quantity:</strong> {booking.quantity} people</p>
    </div>

    <div class="detail-box">
      <h3>👤 Customer Details</h3>
      <p><strong>Name:</strong> {customer.first_name} {customer.last_name}</p>
      <p><strong>Phone:</strong> {customer.phone}</p>
    </div>

    <p>Best regards,<br><strong>The Tangerine Team</strong></p>
  </div>
  <div class="footer">
    <p>© Tangerine Platform. This is an automated message. Please do not reply.</p>
  </div>
</body>
</html>
```

---

## 7. Booking Status Updated

| Field | Value |
|-------|-------|
| **File** | `customers/signals.py` → `send_sms_booking_status()` |
| **Trigger** | `post_save` on `ActivityBooking`, status changes to anything other than `confirmed` or `cancelled` |
| **To** | Customer |
| **From** | ⚠️ hardcoded `bokis.angelov@innovade.eu` — should be `settings.DEFAULT_FROM_EMAIL` |
| **Subject** | `Booking Status Updated` |

**Body:**

```html
<!DOCTYPE html>
<html>
<head>
  <style>
    body { font-family: Arial, sans-serif; color: #333; max-width: 600px; margin: 0 auto; padding: 20px; }
    .header { background: linear-gradient(135deg, #ff6b35, #f7931e); color: white; padding: 24px; border-radius: 10px 10px 0 0; text-align: center; }
    .content { background: #f9f9f9; padding: 24px; border-radius: 0 0 10px 10px; }
    .detail-box { background: white; border: 2px solid #ff6b35; border-radius: 8px; padding: 16px; margin: 16px 0; }
    .footer { text-align: center; font-size: 13px; color: #888; margin-top: 24px; }
  </style>
</head>
<body>
  <div class="header">
    <h1>🔔 Booking Status Updated</h1>
  </div>
  <div class="content">
    <p>Hi <strong>{customer.first_name}</strong>,</p>
    <p>Your booking status has been updated to <strong>{booking.status}</strong>.</p>

    <div class="detail-box">
      <h3>📋 Booking Details</h3>
      <p><strong>Booking ID:</strong> #{booking.id}</p>
      <p><strong>Activity:</strong> {activity_name}</p>
      <p><strong>Date:</strong> {booking.date}</p>
      <p><strong>Quantity:</strong> {booking.quantity} people</p>
      <p><strong>New Status:</strong> {booking.status}</p>
    </div>

    <p>📞 For more information please contact the vendor at <strong>{vendor_phone}</strong>.</p>
    <p>Best regards,<br><strong>The Tangerine Team</strong></p>
  </div>
  <div class="footer">
    <p>© Tangerine Platform. This is an automated message. Please do not reply.</p>
  </div>
</body>
</html>
```

---

## 8. Booking Auto-Expired

| Field | Value |
|-------|-------|
| **File** | `customers/management/commands/expire_pending_bookings.py` → `send_expiry_notification()` |
| **Trigger** | Management command: `python manage.py expire_pending_bookings --notify-customers` |
| **To** | Customer |
| **From** | `settings.DEFAULT_FROM_EMAIL` |
| **Subject** | `Booking Request Expired - {activity_name}` |

**Body:**

```html
<!DOCTYPE html>
<html>
<head>
  <style>
    body { font-family: Arial, sans-serif; color: #333; max-width: 600px; margin: 0 auto; padding: 20px; }
    .header { background: linear-gradient(135deg, #fd7e14, #e83e8c); color: white; padding: 24px; border-radius: 10px 10px 0 0; text-align: center; }
    .content { background: #f9f9f9; padding: 24px; border-radius: 0 0 10px 10px; }
    .detail-box { background: white; border: 2px solid #fd7e14; border-radius: 8px; padding: 16px; margin: 16px 0; }
    .footer { text-align: center; font-size: 13px; color: #888; margin-top: 24px; }
  </style>
</head>
<body>
  <div class="header">
    <h1>⏰ Booking Request Expired</h1>
    <p>The vendor did not respond in time</p>
  </div>
  <div class="content">
    <p>Hi <strong>{customer.first_name or customer.username}</strong>,</p>
    <p>Unfortunately, your booking request has expired because the vendor did not respond within the required timeframe.</p>

    <div class="detail-box">
      <h3>📋 Expired Booking Details</h3>
      <p><strong>Activity:</strong> {activity_name}</p>
      <p><strong>Date:</strong> {booking.date}</p>
      <p><strong>Quantity:</strong> {booking.quantity} people</p>
      <p><strong>Vendor:</strong> {vendor_name}</p>
    </div>

    <p>You can search for alternative activities or try booking with a different vendor on the Tangerine platform.</p>
    <p>We apologise for the inconvenience.</p>
    <p>Best regards,<br><strong>The Tangerine Team</strong></p>
  </div>
  <div class="footer">
    <p>© Tangerine Platform. This is an automated message. Please do not reply.</p>
  </div>
</body>
</html>
```

---

## 9. Vendor Welcome Email

| Field | Value |
|-------|-------|
| **File** | `vendors/signals.py` → `send_vendor_welcome_email()` |
| **Trigger** | `post_save` on `Vendor`, `created=True` |
| **To** | Vendor |
| **From** | `settings.DEFAULT_FROM_EMAIL` ✅ |
| **Subject** | `Welcome to Tangerine - Your Vendor Account is Ready!` |
| **Template** | `templates/vendors/emails/vendor_welcome.html` |

**Body** *(rendered from `vendor_welcome.html`)*:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <style>
    body { font-family: Arial, sans-serif; color: #333; max-width: 600px; margin: 0 auto; padding: 20px; }
    .header { background: linear-gradient(135deg, #ff6b35, #f7931e); color: white; text-align: center; padding: 30px 20px; border-radius: 10px 10px 0 0; }
    .content { background: #f9f9f9; padding: 30px; border-radius: 0 0 10px 10px; }
    .credentials { background: white; border: 2px solid #ff6b35; border-radius: 8px; padding: 20px; margin: 20px 0; }
    .btn { display: inline-block; background: #ff6b35; color: white; padding: 12px 24px; text-decoration: none; border-radius: 5px; margin: 20px 0; }
    .footer { text-align: center; font-size: 14px; color: #666; margin-top: 30px; }
  </style>
</head>
<body>
  <div class="header">
    <h1>🍊 Welcome to Tangerine!</h1>
    <p>Your vendor account has been successfully created</p>
  </div>
  <div class="content">
    <h2>Hello {user.first_name or user.username}!</h2>
    <p>Congratulations! Your vendor account for <strong>{business_name}</strong> has been successfully created on the Tangerine platform.</p>

    <div class="credentials">
      <h3>📋 Your Account Details:</h3>
      <p><strong>Username:</strong> {username}</p>
      <p><strong>Email:</strong> {user.email}</p>
      <p><strong>Business:</strong> {business_name}</p>
      <p><strong>Phone:</strong> {user.phone}</p>  <!-- shown only if set -->
    </div>

    <p>🔐 <strong>Important:</strong> Please keep your login credentials secure.</p>

    <h3>🚀 Next Steps:</h3>
    <ul>
      <li>Complete your vendor profile with additional business information</li>
      <li>Upload your business logo and images</li>
      <li>Add your services and products</li>
      <li>Start connecting with customers!</li>
    </ul>

    <div style="text-align: center;">
      <a href="{login_url}" class="btn">Login to Your Account</a>
    </div>

    <h3>📞 Need Help?</h3>
    <p>If you have any questions or need assistance getting started, please contact our support team.</p>
    <p>Best regards,<br><strong>The Tangerine Team</strong></p>
  </div>
  <div class="footer">
    <p>© Tangerine Platform. All rights reserved.</p>
    <p>This is an automated message. Please do not reply to this email.</p>
  </div>
</body>
</html>
```

---

## 10. Newsletter Broadcast

| Field | Value |
|-------|-------|
| **File** | `newsletter/signals.py` → `send_newsletter_immediately()` |
| **Trigger** | `post_save` on `NewsletterSend`, `created=True` and `send_now=True` |
| **To** | All subscribers in the linked subscription list |
| **From** | `settings.DEFAULT_FROM_EMAIL` ✅ |
| **Subject** | `{newsletter.title}` (dynamic, set in admin) |
| **Body** | `{newsletter.content}` — full HTML content entered via admin CKEditor |

> The body is entirely freeform HTML entered by the admin when creating the newsletter. No fixed template.

---

## Known Issues & TODOs

| # | Issue | File | Fix |
|---|-------|------|-----|
| 1 | All booking emails use hardcoded `from_email="bokis.angelov@innovade.eu"` | `customers/signals.py` | Replace with `settings.DEFAULT_FROM_EMAIL` |
| 2 | New booking email is sent to hardcoded `bokisangelov@gmail.com` instead of the vendor | `customers/signals.py` → `send_sms_new_booking()` | Change recipient to `vendor_email` variable (already available in the function) |
| 3 | Functions named `send_sms_*` but they send emails, not SMS | `customers/signals.py` | Rename to `send_email_*` or implement real SMS via Twilio/etc. |
| 4 | `--notify-customers` flag must be passed manually to the management command | `expire_pending_bookings.py` | Add to a cron job / Celery beat task to run automatically |
| 5 | Newsletter scheduling is not implemented | `newsletter/signals.py` | `TODO: schedule_newsletter(instance)` comment — needs Celery or APScheduler |
