1. What information might this feature expose to Web sites or other parties, and for what purposes is that exposure necessary?

It will expose whether there are PerformanceEntry buffers that are full in a particular global object (Window/Worker). Currently, we buffer these entries so that performance scripts can be loaded lazily while still being able to query data from early on the page load, via the use of the buffered flag. For this purpose we maintain buffers where we store the first performance entry objects, up to a limit. Once this limit is reached, new entries are dropped, i.e. not buffered. The boolean enables the developer to know that their observer callback ran too late, in that it missed some entries. This should be very rare, but we received developer feedback that knowing this would be useful.

2. Is this specification exposing the minimum amount of information necessary to power the feature?

Yes. We could technically expose more information, like which entryTypes were the ones for which an entry was dropped. But in practice most people observe a single entryType per observer, so we went with the simpler boolean design.

3. How does this specification deal with personal information or personally-identifiable information or information derived thereof?

Personal information is not exposed. If a large number of entries would reveal personal information, that would already be possible to detect by looking at the number of entries received from the observer callback.

4. How does this specification deal with sensitive information?

There is no sensitive information being exposed. 

5. Does this specification introduce new state for an origin that persists across browsing sessions?

No.

6. What information from the underlying platform, e.g. configuration data, is exposed by this specification to an origin?

No new configuration data is exposed by this boolean.

7. Does this specification allow an origin access to sensors on a user’s device

No.

8. What data does this specification expose to an origin? Please also document what data is identical to data exposed by other features, in the same or different contexts.

This review is just for the boolean described above. Such boolean is exposed every time the PerformanceObserver callback is called.

9. Does this specification enable new script execution/loading mechanisms?

No.

10. Does this specification allow an origin to access other devices?

No.

11. Does this specification allow an origin some measure of control over a user agent’s native UI?
 No.

12. What temporary identifiers might this this specification create or expose to the web?

None.

13. How does this specification distinguish between behavior in first-party and third-party contexts?

Since buffers and observers are scoped to their relevant global objects, first-party frames have their own data and third-party can gather their own data, but nothing is shared by default. That said, there is no special consideration for the logic of this new boolean in third party contexts.

14. How does this specification work in the context of a user agent’s Private Browsing or "incognito" mode?’

Works just like it would in non-private mode. No special considerations.

15. Does this specification have a "Security Considerations" and "Privacy Considerations" section?

Yes although nothing specific about this new boolean.

16. Does this specification allow downgrading default security characteristics?

No.

17. What should this questionnaire have asked?

Can’t think of anything at the moment.
