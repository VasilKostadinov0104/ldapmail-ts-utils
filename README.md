# LDAP and IMAP utils

A TypeScript utility library for managing LDAP authentication and IMAP email operations with
docker-mailserver. This package simplifies the integration process, providing easy-to-use functions
for secure user authentication via LDAP and efficient email management through IMAP.

Works with:

- https://github.com/ldapmail/ldapmail

## Example

### LDAP actions

For more examples check **tests/ldap.integration.test.ts**

```typescript
const client = LDAPClientUtils.createClient({
    url: 'ldap://127.0.0.1:389',
    bindDN: 'cn=admin,dc=mail,dc=com',
    bindPassword: 'admin$',
    baseDN: 'dc=mail,dc=com',
    crossDomainAliases: true,
    crossDomainGroups: true,
});

await addEmail(client, 'example@example.xxx')
await addAlias(client, 'example@example.xxx', 'alias@example.xxx');
```

### IMAP search for messages

A new client connection should be created for each IMAP action.

```typescript
const IMAPClient = IMAPClientUtils.createClient({
    host: 'localhost',
    port: 143,
    user: 'xxx@xxx',
    password: 'xxx',
    secure: false,
    process_batch_size: 30,
    process_between_batch_size_delay: 100,
});

const result: IMailResult = await searchForMessages(IMAPClient, {
        messageId: '<3b90cef5-xxx-xxx-xxx-xxx@xxx>', // message id
        contains: 'is', // keyword to search
        flag: {
            name: 'answered', // flag name
            value: false, // current flag value
            set: true // set new flag value
        },
    },
    1, // current page 
    20, // messages per page
    'desc' // order by desc (newest -> oldest) or asc (oldest -> newest)
);
```

### Quota
Get quota for email account

```typescript
const IMAPClient = IMAPClientUtils.createClient({
    host: 'localhost',
    port: 143,
    user: 'xxx@xxx',
    password: 'xxx',
    secure: false,
    process_batch_size: 30,
    process_between_batch_size_delay: 100,
});

const quota: IQuotaResult = await getQuota(IMAPClient);
```

### Sync mailboxes

Create and pass target and source IMAP clients to sync mailboxes.

```typescript
await syncMailbox(
    IMAPClientUtils.createClient({
        host: 'localhost',
        port: 143,
        user: 'xxx@xxx',
        password: 'xxx',
        secure: false,
        process_batch_size: 30,
        process_between_batch_size_delay: 100,
    }), // target client
    IMAPClientUtils.createClient({
            host: 'source-host',
            port: 143,
            user: 'source-user-xxx@xxx',
            password: 'xxx',
            secure: false,
            process_batch_size: 30,
            process_between_batch_size_delay: 100,
        }, // source client
        10 //How much messages to fetch at once. The bigger the number, the faster the sync, but the more memory it consumes.
    )
);
```

### Send email

It uses nodemailer

```typescript
    await sendMessage(
    IMAPClientUtils.createClient({
        host: 'localhost',
        port: 465,
        user: 'xxx@xxx',
        password: 'xxx',
        secure: true,
        process_batch_size: 30,
        process_between_batch_size_delay: 100,
    }),
    [
        'bar@localhost',
        'baz@localhost'
    ],
    {
        subject: 'Hello ✔',
        text: 'Hello world?',
        html: '<b>Hello world?</b>'
    },
    true // send individual emails to each recipient from the "to" array
);
```

If you want ot use CC, BCC, attachments or other options from nodemailer you could use directly the
transport.

```typescript
const transport = await getTransport(
    IMAPClientUtils.createClient({
        host: 'localhost',
        port: 465,
        user: 'xxx@xxx',
        password: 'xxx',
        secure: true,
        process_batch_size: 30,
        process_between_batch_size_delay: 100,
    })
);

await transporter.sendMail({...});
```

## Contributors
- https://github.com/Codenetz
- https://github.com/VasilKostadinov0104

## TODO
- Action to return DNS records for MX, DKIM, DMARC, SPF, etc ...
- Email forwarding action
- Set limit for sending emails per hour
- Fix error levels for logs (docker-mailserver, openldap and ldap & imap utils)
