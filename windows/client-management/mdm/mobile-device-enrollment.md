---
title: Mobile device enrollment
description: Learn how  mobile device enrollment verifies that only authenticated and authorized devices can be managed by their enterprise.
ms.assetid: 08C8B3DB-3263-414B-A368-F47B94F47A11
ms.reviewer: 
manager: dansimp
ms.author: dansimp
ms.topic: article
ms.prod: w10
ms.technology: windows
author: manikadhiman
ms.date: 08/11/2017
---

# Mobile device enrollment

Mobile device enrollment is the first phase of enterprise management. The device is configured to communicate with the MDM server using security precautions during the enrollment process. The enrollment service verifies that only authenticated and authorized devices can be managed by their enterprise.

The enrollment process includes the following steps:

1.  Discovery of the enrollment endpoint

    This step provides the enrollment endpoint configuration settings.

2.  Certificate installation

    This step handles user authentication, certificate generation, and certificate installation. The installed certificates will be used in the future to manage client/server Secure Sockets Layer (SSL) mutual authentication.

3.  DM Client provisioning

    This step configures the Device Management (DM) client to connect to a Mobile Device Management (MDM) server after enrollment via DM SyncML over HTTPS (also known as Open Mobile Alliance Device Management (OMA DM) XML).

## Enrollment protocol

There are a number of changes made to the enrollment protocol to better support a variety of scenarios across all platforms. For detailed information about the mobile device enrollment protocol, see [\[MS-MDM\]: Mobile Device Management Protocol](/openspecs/windows_protocols/ms-mdm/33769a92-ac31-47ef-ae7b-dc8501f7104f) and [\[MS-MDE2\]: Mobile Device Enrollment Protocol Version 2]( https://go.microsoft.com/fwlink/p/?LinkId=619347).

The enrollment process involves the following steps:

### Discovery request
   The discovery request is a simple HTTP post call that returns XML over HTTP. The returned XML includes the authentication URL, the management service URL, and the user credential type.

### Certificate enrollment policy
The certificate enrollment policy configuration is an implementation of the MS-XCEP protocol, which is described in \[MS-XCEP\]: X.509 Certificate Enrollment Policy Protocol Specification. Section 4 of the specification provides an example of the policy request and response. The X.509 Certificate Enrollment Policy Protocol is a minimal messaging protocol that includes a single client request message (GetPolicies) with a matching server response message (GetPoliciesResponse). For more information, see [\[MS-XCEP\]: X.509 Certificate Enrollment Policy Protocol](/openspecs/windows_protocols/ms-xcep/08ec4475-32c2-457d-8c27-5a176660a210)

### Certificate enrollment
The certificate enrollment is an implementation of the MS-WSTEP protocol.

### Management configuration
The server sends provisioning XML that contains a server certificate (for SSL server authentication), a client certificate issued by enterprise CA, DM client bootstrap information (for the client to communicate with the management server), an enterprise application token (for the user to install enterprise applications), and the link to download the Company Hub application.

The following topics describe the end-to-end enrollment process using various authentication methods:

-   [Federated authentication device enrollment](federated-authentication-device-enrollment.md)
-   [Certificate authentication device enrollment](certificate-authentication-device-enrollment.md)
-   [On-premise authentication device enrollment](on-premise-authentication-device-enrollment.md)

> [!Note]
> As a best practice, do not use hardcoded server-side checks on values such as:
> -   User agent string
> -   Any fixed URIs that are passed during enrollment
> -   Specific formatting of any value unless otherwise noted, such as the format of the device ID.

## Enrollment support for domain-joined devices

Devices that are joined to an on-premises Active Directory can enroll into MDM via the Work access page in **Settings**. However, the enrollment can only target the user enrolled with user-specific policies. Device targeted policies will continue to impact all users of the device.

## Disable MDM enrollments

In Windows 10 and Windows 11, IT admin can disable MDM enrollments for domain-joined PCs using Group Policy. Using the GP editor, the path is **Computer configuration** &gt; **Administrative Templates** &gt; **Windows Components** &gt; **MDM** &gt; **Disable MDM Enrollment**.

![Disable MDM enrollment policy in GP Editor.](images/mdm-enrollment-disable-policy.png)

Here is the corresponding registry key:

HKLM\SOFTWARE\Policies\Microsoft\Windows\CurrentVersion\MDM

Value: DisableRegistration

## Enrollment scenarios not supported

The following scenarios do not allow MDM enrollments:

- Built-in administrator accounts on Windows desktop cannot enroll into MDM.
- Standard users cannot enroll in MDM. Only admin users can enroll.

## Enrollment error messages

The enrollment server can decline enrollment messages using the SOAP Fault format. Errors created can be sent as follows:

```xml
<s:envelope xmlns:s="http://www.w3.org/2003/05/soap-envelope" xmlns:a="http://www.w3.org/2005/08/addressing">
    <s:header>
        <a:action s:mustunderstand="1">http://schemas.microsoft.com/windows/pki/2009/01/enrollment/rstrc/wstep</a:action>
        <activityid correlationid="2493ee37-beeb-4cb9-833c-cadde9067645" xmlns="http://schemas.microsoft.com/2004/09/servicemodel/diagnostics">2493ee37-beeb-4cb9-833c-cadde9067645</activityid>
        <a:relatesto>urn:uuid:urn:uuid:0d5a1441-5891-453b-becf-a2e5f6ea3749</a:relatesto>
    </s:header>
    <s:body>
        <s:fault>
            <s:code>
                <s:value>s:receiver</s:value>
                <s:subcode>
                    <s:value>s:authorization</s:value>
                </s:subcode>
            </s:code>
            <s:reason>
                <s:text xml:lang="en-us">This User is not authorized to enroll</s:text>
            </s:reason>
        </s:fault>
    </s:body>
</s:envelope>
```

<table>
<colgroup>
<col width="20%" />
<col width="20%" />
<col width="20%" />
<col width="20%" />
<col width="20%" />
</colgroup>
<thead>
<tr class="header">
<th>Namespace</th>
<th>Subcode</th>
<th>Error</th>
<th>Description</th>
<th>HRESULT</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p>s:</p></td>
<td><p>MessageFormat</p></td>
<td><p>MENROLL_E_DEVICE_MESSAGE_FORMAT_ERROR</p></td>
<td><p>Invalid message from the Mobile Device Management (MDM) server.</p></td>
<td><p>80180001</p></td>
</tr>
<tr class="even">
<td><p>s:</p></td>
<td><p>Authentication</p></td>
<td><p>MENROLL_E_DEVICE_AUTHENTICATION_ERROR</p></td>
<td><p>The Mobile Device Management (MDM) server failed to authenticate the user. Try again or contact your system administrator.</p></td>
<td><p>80180002</p></td>
</tr>
<tr class="odd">
<td><p>s:</p></td>
<td><p>Authorization</p></td>
<td><p>MENROLL_E_DEVICE_AUTHORIZATION_ERROR</p></td>
<td><p>The user is not authorized to enroll to Mobile Device Management (MDM). Try again or contact your system administrator.</p></td>
<td><p>80180003</p></td>
</tr>
<tr class="even">
<td><p>s:</p></td>
<td><p>CertificateRequest</p></td>
<td><p>MENROLL_E_DEVICE_CERTIFICATEREQUEST_ERROR</p></td>
<td><p>The user has no permission for the certificate template or the certificate authority is unreachable. Try again or contact your system administrator.</p></td>
<td><p>80180004</p></td>
</tr>
<tr class="odd">
<td><p>s:</p></td>
<td><p>EnrollmentServer</p></td>
<td><p>MENROLL_E_DEVICE_CONFIGMGRSERVER_ERROR</p></td>
<td>The Mobile Device Management (MDM) server encountered an error. Try again or contact your system administrator.</td>
<td><p>80180005</p></td>
</tr>
<tr class="even">
<td><p>a:</p></td>
<td><p>InternalServiceFault</p></td>
<td><p>MENROLL_E_DEVICE_INTERNALSERVICE_ERROR</p></td>
<td><p> There was an unhandled exception on the Mobile Device Management (MDM) server. Try again or contact your system administrator.</p></td>
<td><p>80180006</p></td>
</tr>
<tr class="odd">
<td><p>a:</p></td>
<td><p>InvalidSecurity</p></td>
<td><p>MENROLL_E_DEVICE_INVALIDSECURITY_ERROR</p></td>
<td><p>The Mobile Device Management (MDM) server was not able to validate your account. Try again or contact your system administrator.</p></td>
<td><p>80180007</p></td>
</tr>
</tbody>
</table>

In Windows 10, version 1507, we added the deviceenrollmentserviceerror element. Here is an example:

```xml
<s:envelope xmlns:s="http://www.w3.org/2003/05/soap-envelope" xmlns:a="http://www.w3.org/2005/08/addressing">
    <s:header>
        <a:action s:mustunderstand="1">http://schemas.microsoft.com/windows/pki/2009/01/enrollment/rstrc/wstep</a:action>
        <activityid correlationid="2493ee37-beeb-4cb9-833c-cadde9067645" xmlns="http://schemas.microsoft.com/2004/09/servicemodel/diagnostics">2493ee37-beeb-4cb9-833c-cadde9067645</activityid>
        <a:relatesto>urn:uuid:urn:uuid:0d5a1441-5891-453b-becf-a2e5f6ea3749</a:relatesto>
    </s:header>
    <s:body>
        <s:fault>
            <s:code>
                <s:value>s:receiver</s:value>
                <s:subcode>
                    <s:value>s:authorization</s:value>
                </s:subcode>
            </s:code>
            <s:reason>
                <s:text xml:lang="en-us">device cap reached</s:text>
            </s:reason>
            <s:detail>
                <deviceenrollmentserviceerror xmlns="http://schemas.microsoft.com/windows/pki/2009/01/enrollment">
                    <errortype>devicecapreached</errortype>
                    <message>device cap reached</message>
                    <traceid>2493ee37-beeb-4cb9-833c-cadde9067645</traceid>
                </deviceenrollmentserviceerror>
            </s:detail>
        </s:fault>
    </s:body>
</s:envelope>
```

<table>
<colgroup>
<col width="25%" />
<col width="25%" />
<col width="25%" />
<col width="25%" />
</colgroup>
<thead>
<tr class="header">
<th>Subcode</th>
<th>Error</th>
<th>Description</th>
<th>HRESULT</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p>DeviceCapReached</p></td>
<td><p>MENROLL_E_DEVICECAPREACHED</p></td>
<td><p>The account has too many devices enrolled to Mobile Device Management (MDM). Delete or unenroll old devices to fix this error.</p></td>
<td><p>80180013</p></td>
</tr>
<tr class="even">
<td><p>DeviceNotSupported</p></td>
<td><p>MENROLL_E_DEVICENOTSUPPORTED</p></td>
<td><p>The Mobile Device Management (MDM) server doesn't support this platform or version, consider upgrading your device.</p></td>
<td><p>80180014</p></td>
</tr>
<tr class="odd">
<td><p>NotSupported</p></td>
<td><p>MENROLL_E_NOT_SUPPORTED</p></td>
<td><p>Mobile Device Management (MDM) is generally not supported for this device.</p></td>
<td><p>80180015</p></td>
</tr>
<tr class="even">
<td><p>NotEligibleToRenew</p></td>
<td><p>MENROLL_E_NOTELIGIBLETORENEW</p></td>
<td><p>The device is attempting to renew the Mobile Device Management (MDM) certificate, but the server rejected the request. Check renew schedule on the device.</p></td>
<td><p>80180016</p></td>
</tr>
<tr class="odd">
<td><p>InMaintenance</p></td>
<td><p>MENROLL_E_INMAINTENANCE</p></td>
<td><p>The Mobile Device Management (MDM) server states your account is in maintenance, try again later.</p></td>
<td><p>80180017</p></td>
</tr>
<tr class="even">
<td><p>UserLicense</p></td>
<td><p>MENROLL_E_USER_LICENSE</p></td>
<td><p>There was an error with your Mobile Device Management (MDM) user license. Contact your system administrator.</p></td>
<td><p>80180018</p></td>
</tr>
<tr class="odd">
<td><p>InvalidEnrollmentData</p></td>
<td><p>MENROLL_E_ENROLLMENTDATAINVALID</p></td>
<td><p>The Mobile Device Management (MDM) server rejected the enrollment data. The server may not be configured correctly.</p></td>
<td><p>80180019</p></td>
</tr>
</tbody>
</table>

TraceID is a freeform text node which is logged. It should identify the server side state for this enrollment attempt. This information may be used by support to look up why the server declined the enrollment.

## Related topics

-   [MDM enrollment of Windows-based devices](mdm-enrollment-of-windows-devices.md)
-   [Federated authentication device enrollment](federated-authentication-device-enrollment.md)
-   [Certificate authentication device enrollment](certificate-authentication-device-enrollment.md)
-   [On-premise authentication device enrollment](on-premise-authentication-device-enrollment.md)