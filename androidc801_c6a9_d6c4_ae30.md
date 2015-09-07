**먼저 안드로이드 프로젝트에 라이브러리를 추가한다. **

![](라이브러리추가.PNG)

**MemberVo를 생성해준다. **

```
package com.example.multicall.vo;

import java.util.ArrayList;
import java.util.List;

import com.amazonaws.mobileconnectors.dynamodbv2.dynamodbmapper.*;

@DynamoDBTable(tableName = "Member")
public class Member {

	private String memberID;
	private String memberName;
	private String memberPhoneNumber;
	private ArrayList<String> memberFriendList;
	
	
	@DynamoDBAttribute(attributeName = "FriendList")
	public ArrayList<String> getMemberFriendList() {
		return memberFriendList;
	}

	public void setMemberFriendList(ArrayList<String> memberFriendList) {
		this.memberFriendList = memberFriendList;
	}

	@DynamoDBHashKey(attributeName = "memberID")
	public String getMemberID() {
		return memberID;
	}
	
	public void setMemberID(String memberID) {
		this.memberID = memberID;
	}
	
	@DynamoDBAttribute(attributeName = "MemberName")
	public String getMemberName() {
		return memberName;
	}
	
	public void setMemberName(String memberName) {
		this.memberName = memberName;
	}
	
	@DynamoDBAttribute(attributeName = "MemberPhoneNumber")
	public String getMemberPhoneNumber() {
		return memberPhoneNumber;
	}
	
	public void setMemberPhoneNumber(String memberPhoneNumber) {
		this.memberPhoneNumber = memberPhoneNumber;
	} 
}
```

**내 AWS 계정과 연결할 credential 생성하고 그걸 통해 DynamoDBMapper 객체를 생성한다.**
```
//내 AWS 계정과 연결할 credential 생성 
	    CognitoCachingCredentialsProvider credentialsProvider = new CognitoCachingCredentialsProvider(
	    	    getApplicationContext(),
	    	    "us-east-1:763c7b1e-e427-42b0-8488-feaa887c5613", // Identity Pool ID
	    	    Regions.US_EAST_1 // Region
	    );
	    
	    AmazonDynamoDBClient ddbClient = new AmazonDynamoDBClient(credentialsProvider);
        mapper = new DynamoDBMapper(ddbClient);
```


**AsyncTask를 통해 네트워크 전송한다. **
```
//인터넷이 연결돼 있나 확인 
					if(connect.getNetworkInfo(ConnectivityManager.TYPE_MOBILE).getState() == NetworkInfo.State.CONNECTED 
					|| connect.getNetworkInfo(ConnectivityManager.TYPE_WIFI).getState() == NetworkInfo.State.CONNECTED	
					 ){
						member = new Member();
						member.setMemberID(memberID);
						member.setMemberName(memberName);
						member.setMemberPhoneNumber(memberPhoneNumber);
						new Networking().execute();
					}
```

```
//AsyncTask를 써서 네트워킹 작업 
  	private class Networking extends AsyncTask<URL, Integer, String>{

  		@Override
  		protected void onPreExecute() { 

  			super.onPreExecute();	
  		}
  		
  		@Override
  		protected String doInBackground(URL... params) {

  			String result =null;
  			mapper.save(member);
  			
  			onCancelled();
  			return result;
  		}

	  	@Override
	  	protected void onProgressUpdate(Integer... values) {
	  		super.onProgressUpdate(values);
	  	}
	  	@Override
	  	protected void onPostExecute(String result) {
	  		super.onPostExecute(result);
	  		//doInBackground작업이 끝나면 여기로 와
	  		sp = getSharedPreferences("PreName", MODE_PRIVATE);
			SharedPreferences.Editor editor = sp.edit();
			editor.putString("sp_id", memberID); 
			editor.commit();
			
			Intent intent = new Intent(JoinActivity.this,ListActivity.class);
			intent.putExtra("memberID", memberID);
			startActivity(intent);
    		finish();
    		
	  		//startActivity(new Intent(JoinActivity.this,ListActivity.class));
		}
	  	@Override
	  	protected void onCancelled() {
	  	}
  	}//Networking end
```