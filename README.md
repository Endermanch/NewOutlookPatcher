<h1>NewOutlookPatcher</h1>
<a href="https://discord.gg/gsPcfqHTD2"><img src="https://discordapp.com/api/guilds/1155912047897350204/widget.png?style=shield" alt="Join on Discord"></a>
<p>Disable ads and product placement in new Outlook for Windows app.</p>
<p>Tested on:</p>
<ul>
  <li>Windows 10 Version 21H2 (OS Build 19044.4046)</li>
  <li>Windows 11 Version 23H2 (OS Build 22621.3296)</li>
</ul>
<h2>Donate</h2>
<p><a href="https://github.com/valinet/NewOutlookPatcher?sponsor">PayPal donations</a></p>
<h2>Features</h2>
<ul>
  <li>Disable ad as first item in e-mails list</li>
  <li>Disable lower left corner OneDrive banner</li>
  <li>Disable Word, Excel, PowerPoint, To Do, OneDrive, More apps icons</li>
  <li>Enable F12 Developer Tools</li>
</ul>
<div>
  <img src="https://github.com/valinet/NewOutlookPatcher/assets/6503598/1d70519d-38e2-4663-bee6-46ce3e7dbc90" alt="Product image">
</div>
<h2>How to?</h2>
<ol type="1">
  <li>Download the <a href="https://github.com/valinet/NewOutlookPatcher/releases/latest/download/NewOutlookPatcher.exe">latest release</a>.</li>
  <li>Run <code>NewOutlookPatcher</code>. Outlook will also open automatically in the background.</li>
  <li>Customize the configuration by checking/unchecking individual items.</li>
  <li>Press <code>Patch</code>. The application will elevate itself, close Outlook, apply your setttings and restart Outlook for you.</li>
</ol>
<h2>Why is elevation required?</h2>
<p>New Outlook for Windows is installed in <code>Program Files - WindowsApps</code> which is write-protected for regular users, thus administrative access is required to place the patcher in that folder.</p>
<h2>Uninstalling</h2>
<p>Run <code>NewOutlookPatcher</code>. Uncheck all options. Press <code>Patch</code>. Done.</p>
<h2>Known issues</h2>
<ul>
  <li>Patcher does not start? Install <a href="https://dotnet.microsoft.com/en-us/download/dotnet/thank-you/runtime-desktop-8.0.3-windows-x64-installer">.NET Desktop Runtime 8</a>.</li>
  <li><a href="https://github.com/valinet/NewOutlookPatcher/issues/1">Patcher is x64-only at the moment.</a></li>
  <li><a href="https://github.com/valinet/NewOutlookPatcher/issues/2">Patcher does not read the current configuration at startup, but instead displays suggested default settings.</a></li>
  <li>Patcher cannot run successfully when virtualization-based security is enabled. Open Windows Security - Device security - Core isolation details, disable Memory integrity and reboot. After reboot, execute patcher with desired settings, confirm they work in the Outlook app, and then you can enable back Memory integrity (the custom drivers used for placing files in Outlook's program folder are incompatible with the virtualization-based security feature).</li>
</ul>
<h2>How it works?</h2>
<ul>
  <li>Everything is packed together in a tiny .NET 8-based executable. Required resources are extracted to a temporary folder at runtime.</li>
  <li>Patching Outlook (<code>olk.exe</code>) is done using the now classic <code>dxgi.dll</code> method, exploiting the DLL search order. The project contains a very clean C++ implementation of this technique. This works because the process is not protected, thus being able to load unsigned code.</li>
  <li>The actual patching is done by hooking WebView2 methods, in order to execute scripts that alter the CSS once the main interface loads.</li>
  <li>The main problem for this entire project was, believe it or not, copying the injector to Outlook's program folder. Being in the infamous <code>WindowsApps</code> folder which is thoroughly protected, after a ton of hours researching a user space solution, I resorted to exploiting CVE-2018-19320 to load an own compiled driver which does the copying in kernel space, thus bypassing any ACLs and other user space protections Windows imposes on that folder. CVE-2018-19320 is a technique where a known signed driver that allows for arbitrary kernel memory access via an IOCTL is loaded in the running kernel, and then the exposed IOCTL is used to temporarly disable DSE (driver signature enforcement) in order to further load our custom unsigned driver. DSE has to be enabled back as quickly as possible, as PatchGuard detects the change eventually and bug checks the machine if left like that.</li>
</ul>
<h2>Solution structure</h2>
<p>The Visual Studio solution is divided in 5 projects:</p>
<ul>
  <li>gui: Contains user interface and unpacker logic, C# .NET 8.</li>
  <li>worker: Module that gets loaded by Outlook which injects custom JavaScript and CSS in the user interface.</li>
  <li>installer: Kernel mode driver which copies the worker to Outlook's program folder.</li>
  <li>loader2: Disables DSE and starts loader.exe (based on <a href="https://www.codeproject.com/Articles/5348168/Disable-Driver-Signature-Enforcement-with-DSE-Patc">this</a>).</li>
  <li>loader: Loads custom kernel module (based on <a href="https://github.com/zer0condition/GDRVLoader">this</a>).</li>
</ul>
<p>Successful compilation is only possible for x64 at the moment. Files packed in the final executable are always grabbed from the <code>Release</code> folder, beware when building in <code>Debug</code>.</p>
