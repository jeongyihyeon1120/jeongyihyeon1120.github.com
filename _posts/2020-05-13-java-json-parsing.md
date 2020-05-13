---
layout: post
title:  "Parsing JSON with Java"
date:   2020-5-13 10:20:00 +0700
categories: [Java, freemarker]
---
Json으로 제공된 마스크 api를 자바로 파싱함

**Main code**

```java
package maskApi;

public class Main {

	public static void main(String[] args) {
		
//		mShow(new MaskStore());
//		mShow(new MaskSales());
		mShow(new MaskInfo());
		
	}

	private static void mShow(MaskShow maskShow) {
		// TODO Auto-generated method stub
		maskShow.maskInfoShow();
	}
}

```

마스크 api 에 sales, stores 정보 2개가 있어서  추상클래스 생성

**abstract class**

```java
package maskApi;

public abstract class MaskShow {

	public abstract void maskInfoShow();
	
}
```

#### Store은 병원이나 우체국을 알려주고 주소를 알려준다.

**MaskStoreInput**

```java
package maskApi;

public class InputStore {
	public String addr;
	public String code;
	public String lat;
	public String lng;
	public String name;
	public String type;
	public String getAddr() {
		return addr;
	}
	
	public InputStore(String type, String name, String addr, String code, String lat, String lng) {
		this.type = type;
		this.name = name;
		this.addr = addr;
		this.code = code;
		this.lat = lat;
		this.lng = lng;
	}
	
	public void setAddr(String addr) {
		this.addr = addr;
	}
	public String getCode() {
		return code;
	}
	public void setCode(String code) {
		this.code = code;
	}
	public String getLat() {
		return lat;
	}
	public void setLat(String lat) {
		this.lat = lat;
	}
	public String getLng() {
		return lng;
	}
	public void setLng(String lng) {
		this.lng = lng;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public String getType() {
		return type;
	}
	public void setType(String type) {
		this.type = type;
	}
	
}
```

**MaskStore**

```java
package maskApi;

import static spark.Spark.get;
import static spark.Spark.modelAndView;
import static spark.Spark.port;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.Map;

import org.json.simple.JSONArray;
import org.json.simple.JSONObject;
import org.json.simple.parser.JSONParser;
import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;

import com.google.gson.Gson;
import com.google.gson.GsonBuilder;

import freemarker.FreeMarkerTemplateEngine;

public class MaskStore extends MaskShow {
	@Override
	public void maskInfoShow() {
		port(8088);
		ArrayList<InputStore> jArr = new ArrayList<InputStore>();
		Map<String, Object> attributes = new HashMap<>();
		get("/mask/store/:page", (request, response) -> {
			InputStore is = null;
			ArrayList<String> storeInfos = new ArrayList<String>();
			Document doc = Jsoup.connect(
					"https://8oi9s0nnth.apigw.ntruss.com/corona19-masks/v1/stores/json?page=" + request.params("page"))
					.ignoreContentType(true).get();
			String body = doc.getElementsByTag("body").text();
			try {
				JSONParser parser = new JSONParser();
				Object obj = parser.parse(body);
				JSONObject jsonObj = (JSONObject) obj;
				JSONArray storeInfoArray = (JSONArray) jsonObj.get("storeInfos");
				if (storeInfoArray.size() == 0) return modelAndView(null, "test.ftl");
				for (int i = 0; i < storeInfoArray.size(); i++) {
					// 배열 안에 있는것도 JSON형식 이기 때문에 JSON Object 로 추출
					JSONObject storeObject = (JSONObject) storeInfoArray.get(i);
//					System.out.println(storeObject.get("code"));
//					System.out.println(page);
					if(storeObject.get("lat") == null&&storeObject.get("lng") == null) {
						storeInfos.add(new GsonBuilder().serializeNulls().create()
								.toJson(new InputStore((String)storeObject.get("type"),
										(String)storeObject.get("name"), (String)storeObject.get("addr"),
										storeObject.get("code").toString(), null,
										null)));
						continue;
					}
					
					storeInfos.add(new GsonBuilder().serializeNulls().create()
							.toJson(new InputStore((String)storeObject.get("type"),
									(String)storeObject.get("name"), (String)storeObject.get("addr"),
									storeObject.get("code").toString(), Double.toString((double) storeObject.get("lat")),
									Double.toString((double) storeObject.get("lng")))));
					if(storeObject.get("code").equals("31869475")) System.out.println(1);
				}

			} catch (Exception e) {
				e.printStackTrace();
			}
			for (int i = 0; i < storeInfos.size(); i++) {
				is = new Gson().fromJson(storeInfos.get(i), InputStore.class);
				jArr.add(new InputStore(is.type, is.name, is.addr, is.code, is.lat, is.lng));
			}
			attributes.put("message", jArr);
			return modelAndView(attributes, "test.ftl");
		}, new FreeMarkerTemplateEngine());
	}
}
```

#### Sales은 마스크의 재고를 확인한다.

**MaskSalesInput**

```java
package maskApi;

public class InputSales {
	public String code;
	public String created_at;
	public String remain_stat;
	public String stock_at;
	
	public InputSales(String code, String created_at, String remain_stat, String stock_at) {
		this.code = code;
		this.created_at = created_at;
		this.remain_stat = remain_stat;
		this.stock_at = stock_at;
	}
	
	public String getCode() {
		return code;
	}
	public void setCode(String code) {
		this.code = code;
	}
	public String getCreated_at() {
		return created_at;
	}
	public void setCreated_at(String created_at) {
		this.created_at = created_at;
	}
	public String getRemain_stat() {
		return remain_stat;
	}
	public void setRemain_stat(String remain_stat) {
		this.remain_stat = remain_stat;
	}
	public String getStock_at() {
		return stock_at;
	}
	public void setStock_at(String stock_at) {
		this.stock_at = stock_at;
	}
}
```

**MaskSales**

```java
package maskApi;

import static spark.Spark.get;
import static spark.Spark.modelAndView;
import static spark.Spark.port;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.Map;

import org.json.simple.JSONArray;
import org.json.simple.JSONObject;
import org.json.simple.parser.JSONParser;
import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;

import com.google.gson.Gson;
import com.google.gson.GsonBuilder;

import freemarker.FreeMarkerTemplateEngine;

public class MaskSales extends MaskShow {

	@Override
	public void maskInfoShow() {
		port(45678);
		ArrayList<Object> jArr = new ArrayList<Object>();
		Map<String, Object> attributes = new HashMap<>();
		get("/mask/sales/:page", (request, response) -> {
			InputSales is = null;
			ArrayList<String> salesInfos = new ArrayList<String>();
			Document doc = Jsoup.connect(
					"https://8oi9s0nnth.apigw.ntruss.com/corona19-masks/v1/sales/json?page=" + request.params("page"))
					.ignoreContentType(true).get();
			String body = doc.getElementsByTag("body").text();
			try {
				JSONParser parser = new JSONParser();
				Object obj = parser.parse(body);
				JSONObject jsonObj = (JSONObject) obj;
				JSONArray salesInfoArray = (JSONArray) jsonObj.get("sales");
				if (salesInfoArray.size() == 0)
					return modelAndView(null, "test.ftl");
				for (int i = 0; i < salesInfoArray.size(); i++) {
					// 배열 안에 있는것도 JSON형식 이기 때문에 JSON Object 로 추출
					JSONObject salesObject = (JSONObject) salesInfoArray.get(i);
					salesInfos.add(new GsonBuilder().serializeNulls().create()
							.toJson(new InputSales(salesObject.get("code").toString(),
									(String) salesObject.get("created_at"), (String) salesObject.get("remain_stat"),
									(String) salesObject.get("stock_at"))));
					if (salesObject.get("code").equals("31869475"))
						System.out.println(1);
				}

			} catch (Exception e) {
				e.printStackTrace();
			}
			for (int i = 0; i < salesInfos.size(); i++) {
				is = new Gson().fromJson(salesInfos.get(i), InputSales.class);
				jArr.add(new InputSales(is.code, is.created_at, is.remain_stat, is.stock_at));
			}
			attributes.put("message", jArr);
			return modelAndView(attributes, "test1.ftl");
		}, new FreeMarkerTemplateEngine());
	}

}
```

#### Sales과 Store을 합쳐서 코드가 같은것 끼리 묵어 재고와 주소를 확인시켜준다. (본인이 코드 합침)

**MaskInfoInput**

```java
package maskApi;

public class InputMaskShow {
	public String name;
	public String addr;
	public String remain_stat;
	
	public InputMaskShow (String name, String addr, String remain_stat) {
		this.name = name;
		this.addr = addr;
		this.remain_stat = remain_stat;
	}
	
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public String getAddr() {
		return addr;
	}
	public void setAddr(String addr) {
		this.addr = addr;
	}
	public String getRemain_stat() {
		return remain_stat;
	}
	public void setRemain_stat(String remain_stat) {
		this.remain_stat = remain_stat;
	}
}
```

**MaskInfo**

```java
package maskApi;

import static spark.Spark.get;
import static spark.Spark.modelAndView;
import static spark.Spark.port;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.Map;

import org.json.simple.JSONArray;
import org.json.simple.JSONObject;
import org.json.simple.parser.JSONParser;
import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;

import com.google.gson.Gson;
import com.google.gson.GsonBuilder;

import freemarker.FreeMarkerTemplateEngine;

public class MaskInfo extends MaskShow {

	@Override
	public void maskInfoShow() {
		port(8088);
		ArrayList<Object> jArr = new ArrayList<Object>();
		Map<String, Object> attributes = new HashMap<>();
		get("/mask/show", (request, response) -> {
			InputMaskShow ims = null;
			ArrayList<String> salesInfos = new ArrayList<String>();
			ArrayList<String> storeInfos = new ArrayList<String>();
			ArrayList<String> maskInfos = new ArrayList<String>();
			int page = 0;
			while (true) {
				System.out.println("sales반복문");
				page++;
				System.out.println("페이지: " + page);
				Document docSales = Jsoup
						.connect("https://8oi9s0nnth.apigw.ntruss.com/corona19-masks/v1/sales/json?page=" + page)
						.ignoreContentType(true).get();
				String salesBody = docSales.getElementsByTag("body").text();
				try {
					JSONParser parser = new JSONParser();
					Object objSales = parser.parse(salesBody);
					JSONObject jsonObjSales = (JSONObject) objSales;
					JSONArray salesInfoArray = (JSONArray) jsonObjSales.get("sales");
					if (salesInfoArray.size() == 0 && page != 1) {
						System.out.println("멈춤");
						break;
					}
					for (int i = 0; i < salesInfoArray.size(); i++) {
						// 배열 안에 있는것도 JSON형식 이기 때문에 JSON Object 로 추출
						JSONObject salesObject = (JSONObject) salesInfoArray.get(i);
						salesInfos.add(salesObject.get("code").toString());
						salesInfos.add((String) salesObject.get("remain_stat"));
					}
				} catch (Exception e) {
					// TODO: handle exception
				}
			}
			page = 0;
			while (true) {
				System.out.println("sales반복문");
				page++;
				System.out.println("페이지: " + page);
				Document docStore = Jsoup
						.connect("https://8oi9s0nnth.apigw.ntruss.com/corona19-masks/v1/stores/json?page=" + page)
						.ignoreContentType(true).get();
				String storeBody = docStore.getElementsByTag("body").text();
				try {
					
					JSONParser parser = new JSONParser();
					Object objStore = parser.parse(storeBody);
					JSONObject jsonObjStore = (JSONObject) objStore;
					JSONArray storeInfoArray = (JSONArray) jsonObjStore.get("storeInfos");
					if (storeInfoArray.size() == 0 && page != 1) {
						System.out.println("멈춤");
						break;
					}
					for (int i = 0; i < storeInfoArray.size(); i++) {
						JSONObject storeObject = (JSONObject) storeInfoArray.get(i);
						storeInfos.add(storeObject.get("code").toString());
						storeInfos.add((String) storeObject.get("name"));
						storeInfos.add((String) storeObject.get("addr"));
					}

				} catch (Exception e) {
					e.printStackTrace();
				}
			}
			for (int i = 0; i < salesInfos.size(); i += 2) {

				maskInfos.add(new GsonBuilder().serializeNulls().create()
						.toJson(new InputMaskShow(storeInfos.get(storeInfos.indexOf(salesInfos.get(i)) + 1),
								storeInfos.get(storeInfos.indexOf(salesInfos.get(i)) + 2), salesInfos.get(i + 1))));
			}
			for (int i = 0; i < maskInfos.size(); i++) {
//				System.out.println(salesInfos.get(j));
//				System.out.println(salesInfos.get(j + 1));
				ims = new Gson().fromJson(maskInfos.get(i), InputMaskShow.class);
				if(ims.remain_stat == null) {
					ims.remain_stat = "null값 입니다.";
				}
				jArr.add(new InputMaskShow(ims.name, ims.addr, ims.remain_stat));
			}
			attributes.put("message", jArr);
			return modelAndView(attributes, "maskShow.ftl");
		}, new FreeMarkerTemplateEngine());
	}

}
```
