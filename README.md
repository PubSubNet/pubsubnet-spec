# PubSubNet spec

architecture description and specification of the PubSubNet content networking protocol, extending WebSub's Publish/Subscribe model

currently this document has ideas and non-normative descriptions, taken from my notes, of the very first revision of the specification and / or protocol. 

**briefly**: PubSubNet aims to replace Twitter, Facebook, and similar centralised for-profit social media, by putting users in control of their social network. Personal data is stored on your device locally, and only content you choose is uploaded to the Hubs for others to see.

---

  - user-facing arch:
    - each user has a board (wall or timeline like)
    - the board can have documents and statusupdate-likes on it
    - files and documents are locally hosted, then losslessly compressed and sent to the hub(s)
    - users can download entire userboards or specific items from hubs
    - board items are timestamped with their creation, their upload-by-owner ts, their first-seen-by ts, and their seen-by-user ts
    - boards and their items can be scheduled to exist and self destruct
    - boards may be owned and represent a human being, but they can represent another entity too
    - non-user boards can be owned / operated by multiple users, who can moderate it and allow non-mods to add items

    - when a user adds an item to a board
      - the client collects all the resources required by the board (link resolution, media, timestamps)
      - the message is base64-encoded, losslessly compressed (bz2, 7z, gz?), and then base64-encoded again 
      - then it is signed / (maybe) fully encrypted, compressed, base64-encoded again
      - it is published to hub(s) over HTTPS/2 w/ metadata
        tags, (maybe) index of its plaintext, timestamps, hash, signature, username of creator
      - subscribers recieve the message when they query the hub, but may need a key to decrypt it

    - boards may have a theme, and clients can also override all themes (replacing images, dark mode, no custom user themes etc)

    - when a user deletes or hides an item from a board
      - the client publishes that the resource is deleted 
      - the hub agrees, and trashes its copy of the resource 
      - holding clients should trash their copy but it is impossible to force this
      - publishers are notified about clients who don't subscribe to deletions

  - security/privacy: 
    - each user has a private key assigned to them; it can be regenerated (this does not rewrite history though)
    - an assymetric EPKE scheme is used to sign media, end to end encryption is carried out another way
    - scheduled items are simply not downloaded by clients until their scheduled time
    - self-destructing items are eventually deleted by the creating client, and other clients should make such items disappear 
    - (but it is impossible to ensure that clients will, so instead the self-destruct feature comes with the caveat that some users may keep the post)
    - the same is true for deletion, clients are told about the deletion but they may maliciously choose not to delete it
    - users are warned that not subscribing to deletions / destructions is considered rude and an invasion of privacy

    - decryption private key distribution will be recommended to be done over Tox, Messenger Secret Conversations or some other secure channel

- Tox-based personal messaging system with Tox features (?)
  - no, just recommend use tox for privatemessaging

# architecture
  - auth is always client-side, hub doesn't do auth
  - auth can be non-obscuring because we are only protecting against client-side malicious attackers
  - web-based client?
    - need HTML5 localstorage, and a lot of it
    - need ability to upload HDD media into browser localstorage, to publish
    - probably need WebSockets for HTTP/2 drop-in?
  - native
    - cache, configuration, and security keys are stored separately 
    - cache and security keys are encrypted by the app password, and are decrypted in memory

  - all participants in a network need to declare: 
    - spec / protocol revision they implement
    - hub/client semantic version (version+patchlevel)

  - cache is:
    - of two kinds, volatile and permanent
    - board cache
      - current user
      - owned / operated boards
      - subscribed boards
    - userdata cache
      - names, themes
        - current user
        - other users
    - "personal" data
      - first-seen timestamps of items

## security keys
  - keywords: EPKE, diffie-hellman, forward secrecy, non-repudiation
  - own:
    - client password hash
    - signing keys
    - private encryption keys
    - shared decryption keys
  - others:
    - signing keys
    - shared decryption keys
  - client has a local passphrase that decrypts local data 
    - the cutting edge hates passwords but we are not trying to revolutionise authentication
    - this can be changed?, but cannot be recovered
  - client has some sort of asymmetric encryption key 
    - forward secrecy might be impossible with a simple scheme
  - client has a digital signature which can't be regenerated without a new identity  
    - easy to make password static so that it is tied to digsig

  - probably rely on GnuPG (libgcrypt)

## configuration
  - theme overrides, universal dark mode, etc
  - subscriptions / subscribers?
  - whether to (down)load media images, videos, audio, etc
  - what kind of timestamps to show 
  - whether to verify signatures / hashes
  - whether to sign / encrypt messages
  - list of hub root endpoints to use
    
## hub
  - implements at least WebSub 0.4 spec and additional features 
  - doesn't know anything (keys, users, decompression)
  - is only a dumb container for data 
  - responds to different kinds of publish requests
  - supports HTTP/2
  - trashes content after some (variable) timeout (max 30 days?)

## client 
  - is simultaneously a publisher and subscriber
  - supports HTTP/2
  - does all of the heavy lifting, encoding, signing/encrypting, presentation, validation / verification
  - multi-user

## board
  - named, tagged?, encrypted?, ordered? collection of items
  - represented as a feed within WebSub
  - indexable to the extent that its items are
  - owner (admin) > operator (moderator) > user (item/event publisher) > observer (read-only)
  - can have 1 or many owners and operators, but will always have at least 1

  - personal ("user profile")
    - 1 combined owner/operator
    - may be publicly observable / eventable
    - ideally represent an individual human being or robot
    - user's own board specifically may not have ONs/OPs added
    - may be encrypted, whole or per-item
  - private
    - many owners + operators
    - may be publicly observable
    - not public to event publishing 
    - may represent an organisation's internal comms
    - may be encrypted, whole or per-item (how?)
  - public
    - many owners + operators
    - must be publicly observable
    - may be publicly eventable
    - may represent an organisation's public comms
    - cannot be encrypted 

## item
  - any top-level resource: text, image+gif, audio, video, linkshare, poll / form, generalised file, etc
  - may "contain" an event trigger like a @user or @id "ping", or some other thing
    - users @mentioned / tagged like this are able only to remove the mention
  - metadata:
    - timestamps: CReation, UpLoad, FirstSeen, FakeTime
    - schedule timestamps: AppearAt, DestructAt
    - signatures: hash, digsig of user
    - ids: UUUID, username, pubkey
    - priority scheme (high / airmail priority things are published / retrieved first, but there is a subscriber (client) observed quota)
    - non-critical content metadata: revision history, tags, word index, language?
      - for files: original filename, content encoding
    - NOT: compression strategy, content lengths, MIME types, etc, those likely belong on a different non-content level?

  - items and their original metadata can be re-uploaded by anyone
  - the re-upload is not transparent and shows that it is not the original upload
      (i.e. its uploader differs from its author)

  - item content can be in any order (image/png, text/plain, audio/flac, text/html, application/pdf...)
    - this presents small challenges for compression and full encoding/encryption
    - content pieces are individually [b64'd, compressed, b64'd again] then glued with record separators, b64'd again, then maybe encrypted
    - metadata will have to be duplicated or stored "separate" in another part 

  - some items will be rejected by some hubs for being too long
  - hubs "quietly" low-priority tell subscribers and publishers that they are running out of space, if they are 

## event
  - something which "happens" on or to an item
  - "commentary", "reaction", a re-upload of someone else's item, etc
  - events are published "backwards" up the vine, from subscriber to original publisher

  - to see someone's board items in your feed(?) 
    - subscribe to them to get their items / board
    - including encrypted items
  - "add" / "friend" them, which both users must approve 
    - may choose to share private keys

  - user:
    - an identity, not connected to one client
    - username, u3id (universally unique user identifier), "pretty" display name?, public key, (secretly) a private key
    - usernames / dispnames are not required to be unique, but u3ids and public keys are
    - user/dispnames can be changed at will, keys can be changed sometimes, but u3ids cannot be changed
    - users are cheap and can be readily discarded because they are only a "tag" on objects, like boards and items 

# questions / challenges
  - base64 tends to grow quite large 137%, but python pickle is not universal
  - key sharing / expiration (managed by application)
    - public signing keys can be stored on hubs, but private keys cannot be traded over hubs
    - there is no direct communication between clients, though, so TLS-like handshakes will not work
  - metadata duplication / location
  - message header / format? 
  - encryption of large things 
  - minor signing + hashing challenges
  - multi-device access is an impossibility without central server
    - suggest use syncthing or other syncing / cloud service (local data is encrypted anyway)
  - things encrypted / signed with old public keys cannot be updated to use the new public key
    - it is impossible, but it is also semantically impractical / insecure 
  - design some kind of `pubsubnet://` uniform resource identification / location scheme
    - for boards, items, users, events etc
    - different kinds to refer to local vs arbitrary remote hub resources
  - subscribing to a board should give you its old posts (without downloading the entire board history) 
  - real concrete date indexing, like Facebook, Twitter and Instagram don't
