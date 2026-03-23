+++
title =  "Footer"
type = "footer"
draft = false
+++


{{< contact-section
    title="Reach out" 
    contact_form_name="Your name"
    contact_form_email="Your e-mail"
    contact_form_message="Your text"
    contact_form_phone="Your phone"
    contact_button="Send message"
    contact_phone_title="My phone"
    contact_phone_number="<a href='tel:+6285121043828'>6285 1210 43828</a>"
    contact_email_title="My mail"
    contact_email_email="<a href='mailto:contact@ferico.my.id'>contact@ferico.my.id</a>"
    contact_address_title="My location"
    contact_address_address="ID Indonesia"
    form_action="https://formspree.io/f/mdawrwpv"
    form_method="POST"
    contact_form_rows="3"
>}}

{{< newsletter-section 
    newsletter_title="Stay updated"
    newsletter_placeholder="Enter your email"
    newsletter_button="Subscribe"
    newsletter_success_message="Thank you for subscribing!"
    newsletter_error_message="Something went wrong, please try again."
    newsletter_note="We respect your privacy and won't share your data."
    form_action="/"
    form_method="POST"
>}}


{{< text-section
title="Extra footer content"
centered="true"
>}}
Additional content added after the `section` blocks. 

Here you could freestyle, add other shortcodes, ...  Or just let the content empty, and rely on the shortcode sections alone.

To make the text nicely wrapped in the footer, the shortcode `text-section` is used.
{{< /text-section >}}