# Fiddler.4.OnWebSocketMessage
OnWebSocketMessage handler for Fiddler 4 / Fiddler 5

Based on https://github.com/engineforce/InspectWebSocket

# Step 1. Add Custom Funtions: HexToString, OnWebSocketMessage, Fill...Column

![image](https://user-images.githubusercontent.com/7660287/143252893-61ea0ac7-3a1b-44be-87b9-3bbaf77ed582.png)

```c#

	// Extract Hex to String.
	// E.g., 7B-22-48-22-3A-22-54-72-61-6E to {"H":"TransportHub","M":"
	//
	static function HexToString(sourceHex: String)
	{
		sourceHex = sourceHex.Replace("-", "");
    
		var sb = new System.Text.StringBuilder();
    
		for (var i = 0; i < sourceHex.Length; i += 2)
		{
			var hs = sourceHex.Substring(i, 2);
			sb.Append(Convert.ToChar(Convert.ToUInt32(hs, 16)));
		}
    
		var ascii = sb.ToString();
		return ascii;

	}
    //
    // Listen to WebSocketMessage event, and add the socket messages
    // to the static queue.
    //
	static function OnWebSocketMessage(oMsg: WebSocketMessage)
	{
		var msgStr = oMsg.ToString();
		var head = msgStr.Split("\n")[0];
		var masking = msgStr.Split("\n")[4].Substring(9);
		var wsFrom = oMsg.IsOutbound ? "Client" : "Server";
		var wsSession = head.Split(".")[0];
		var wsSocket =  head.Split("'")[1];
		
		var start = String.Format(
			"POST http://ws/{0} HTTP/1.1\n" +
			"User-Agent: Fiddler\n" +
			"Content-Type: application/json; charset=utf-8\n" +
			"Host: ws.{1}\n" + 
			"Content-Length: 0\n\n",
			wsFrom + "/" + oMsg.FrameType + "/" + oMsg.ID, wsSession + "." + wsSocket.Split("#")[1]);
		
		var session = FiddlerApplication.oProxy.SendRequest(start, null);
		if( session != null ) {
			session.UNSTABLE_SetBitFlag(SessionFlags.IsWebSocketTunnel, true);
			session.UNSTABLE_SetBitFlag(SessionFlags.RequestGeneratedByFiddler, true);
			session.UNSTABLE_SetBitFlag(SessionFlags.ResponseGeneratedByFiddler, true);
			session.utilSetRequestBody(oMsg.PayloadAsString());
			session.oRequest.headers.Add("wsIsOutbound", oMsg.IsOutbound);
			session.oRequest.headers.Add("wsFrom", wsFrom);
			session.oRequest.headers.Add("wsiCloseReason", oMsg.iCloseReason);
			session.oRequest.headers.Add("wsID", oMsg.ID);
			session.oRequest.headers.Add("wsFrameType", oMsg.FrameType);
			session.oRequest.headers.Add("wsIsFinalFrame", oMsg.IsFinalFrame);
			session.oRequest.headers.Add("wsPayloadLength", oMsg.PayloadLength);
			session.oRequest.headers.Add("wsdtDoneRead", oMsg.Timers.dtDoneRead.ToString("hh:mm:ss.fff"));
			session.oRequest.headers.Add("wsWasAborted", oMsg.WasAborted);
			session.oRequest.headers.Add("wsPayload", oMsg.PayloadAsString());
			session.oRequest.headers.Add("wsSession", wsSession);
			session.oRequest.headers.Add("wsSocket", wsSocket);
			session.oRequest.headers.Add("wsMasking", masking);
			if (oMsg.FrameType != WebSocketFrameTypes.Text)
			{
				session.oRequest.headers.Add("wsPayloadPlain", HexToString(oMsg.PayloadAsString()));
			}	
		}        
	}
	public static BindUIColumn("wsFrom", 90)
	function Fill_wsFromColumn(oS: Session): String {
		if (oS.RequestHeaders.Exists("wsFrom"))
			return oS.RequestHeaders.AllValues("wsFrom");
		else
			return "";
	}
	public static BindUIColumn("wsFrameType", 90)
	function Fill_wsFrameTypeColumn(oS: Session): String {
		if (oS.RequestHeaders.Exists("wsFrameType"))
			return oS.RequestHeaders.AllValues("wsFrameType");
		else
			return "";
	}
	public static BindUIColumn("wsPayload", 90)
	function Fill_wsPayloadColumn(oS: Session): String {
		if (oS.RequestHeaders.Exists("wsPayload"))
			return oS.RequestHeaders.AllValues("wsPayload");
		else
			return "";
	}
	public static BindUIColumn("wsPayloadPlain", 90)
	function Fill_wsPayloadPlainColumn(oS: Session): String {
		if (oS.RequestHeaders.Exists("wsPayloadPlain"))
			return oS.RequestHeaders.AllValues("wsPayloadPlain");
		else
			return "";
	}
	
 ```
 
 ![image](https://user-images.githubusercontent.com/7660287/143253164-7f0e674a-7237-4279-99d5-680a1dbca72c.png)


 # Step 2. Setup Fiddler -> AutoResponder (Optional)
 
 For `http://ws`
 
 ![image](https://user-images.githubusercontent.com/7660287/143252732-0062cac7-b073-441e-97d5-5002c748da05.png)

# Step 3. Setup WebBrowser work with Fiddler

# View WebSocket messages in Fiddler

Test with https://www.piesocket.com/websocket-tester

![image](https://user-images.githubusercontent.com/7660287/143254113-c9b17884-f97a-4852-8254-884c270384b1.png)
