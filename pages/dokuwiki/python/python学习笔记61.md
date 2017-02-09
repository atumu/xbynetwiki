title: python学习笔记61 

#  Python开源库之EMail操作库 
Envelopes
Mailing for human beings.
官网:http://tomekwojcik.github.io/envelopes/index.html
from envelopes import Envelope, GMailSMTP

```

envelope = Envelope(
    from_addr=(u'from@example.com', u'From Example'),
    to_addr=(u'to@example.com', u'To Example'),
    subject=u'Envelopes demo',
    text_body=u"I'm a helicopter!"
)
envelope.add_attachment('/Users/bilbo/Pictures/helicopter.jpg')

# Send the envelope using an ad-hoc connection...
envelope.send('smtp.googlemail.com', login='from@example.com',
              password='password', tls=True)

# Or send the envelope using a shared GMail connection...
gmail = GMailSMTP('from@example.com', 'password')
gmail.send(envelope)

```
