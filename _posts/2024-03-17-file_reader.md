---
layout: post
title: 리액트 파일 읽기(fileReader), 파일 객체 만들기(new File)
subtitle: 파일 읽.수.만 📂
categories: file
tags: [react, file, blob]
image: /assets/images/20240317_file.png
---

#### 들어가며😃

---

프로젝트에서 yaml 파일을 읽고, 수정하고, 만들어보았다. 팝업을 호출할 때 등록한 파일을 넘겨주고, 팝업은 전달받은 파일을 해석하여 이름과 내용을 에디터에 보여준다. 그리고 사용자가 내용을 수정하고 팝업을 닫으면 변경된 내용을 바탕으로 파일을 새로 만드는 과정이였다. 해당 내용을 처음 접했을때는 파일을 만드려면 라이브러리를 사용해야하는 것인가? 라고 생각할 만큼 파일 해석, 생성 과정에 초면이였다. 결론은 new File 이라는 파일 객체 생성으로 간단하게 구현할 수 있었고, 이 과정에서 Blob 이라는 타입을 처음 접하게 되었다. 그래서 파일 관련 로직과 Blob 타입에 대해 간단하게 정리하고 넘어가려한다.

<br/><br/>

#### 1. 업로드 인자 살펴보기 👀

---

우선 나는 Antd의 Upload 컴포넌트를 사용하여 파일 내용을 살폈다.

```typescript
const uploadProps: UploadProps = {
  name: "file",
  multiple: false,
  accept: "application/x-yaml",
  beforeUpload: () => false,
  onChange: (info) => {
    console.log(info);
  },
};
```

<img width="400" alt="file log" src="https://github.com/ju-ju2/ju-ju2.github.io/assets/71650663/01f33802-80c2-4e1e-9591-2f4cf1f548b9">  
<br/>

위의 이미지와 같이 파일을 올리면 file과 fileList가 들어온다. 파일 내용을 읽기위해서는 `originFileObj`가 필요하기 때문에 두 인자중에 fileList를 사용해야한다.
<br/>

```typescript
  const [fileName, setFileName] = useState<string>("");
  const [file, setFile] = useState<RcFile>();

    onChange: (info) => {
      if (info.file.status !== "done") {
        const file = info.fileList[0];
        const fileName = file.name;
        setFileName(fileName);
        setFile(file.originFileObj);
      }
    },
```

파일 내용을 바탕으로 파일의 이름과 originFileObj 값을 상태값으로 저장한다.
<br/><br/>

#### 2. 파일 읽기 📖

---

**1 .FileReader 객체 생성**
`fileReader = new FileReader()`
<br/>
팝업에서 전달받은 파일을 해석해야한다. 그러기 위해 `FileReader` 라는 객체를 사용할건데, `FileReader`는 **File** 혹은 **Blob** 객체를 이용해 파일의 내용을 읽고 사용자의 컴퓨터에 저장하는 것을 가능하게 해준다.

**2 .readAsText**
<br/>
`fileReader.readAsText(File 또는 Blob)`  
readAsText를 통해 파일 객체의 내용을 텍스트로 읽는다.

**3 .onload, result**
<br/>
`fileReader.onload = () => {}`  
파일 읽기 작업이 완료되면 onload 기능이 실행된다. 그때 그 결과(result)를 내가 원하는 상태에 저장한다.

```typescript
useEffect(() => {
  if (file) {
    const getFile = () => {
      let fileReader = new FileReader();
      fileReader.readAsText(file);
      fileReader.onload = () => {
        if (typeof fileReader.result === "string") {
          setYamlValue(fileReader.result);
        }
      };
    };
    getFile();
  }
}, [file]);
```

<br/><br/>

#### 3. 파일 생성하기 📂

---

팝업 내부에서 파일의 이름과 내용을 수정했고, 이 내용을 다시 새로운 파일로 생성해줘야한다.

위에서 FileReader 객체를 생성해준 것 처럼 같은 방식으로 File 객체를 생성해준다. 문법와 인자는 아래과 같다.

`new File(fileBits, fileName, options);`

**fileBits** : 파일의 내용이며 BufferSource / Blob / string 타입이여야한다.

**fileName** : 만들어낼 파일의 이름을 지정

**options** : 옵션 값이며, type/endings/lastModified 을 설정할 수 있다.

```typescript
const handleConvertFile = (): File => {
  const file = new File([yamlValue], fileName, {
    type: "application/x-yaml",
  });

  return file;
};
```

위에서 만든 파일의 로그를 찍어보면 아래 이미지와 같이 파일이 잘 생성된 것을 확인할 수 있다.😎

<img width="400" alt="image" src="https://github.com/ju-ju2/ju-ju2.github.io/assets/71650663/e74d69c3-7a04-41d8-ae06-1cf8d0af7e23">

<br/><br/><br/><br/>

---

처음 로직을 구성할때 Antd 에서 제공하는 타입과 기본타입이 안맞아 애를 먹었었는데 막상 최종 결과는 딱히 상관없는 관계였다,,흑 언제 타입 마스터가 될 수 있을런지😵‍💫😵‍💫 여튼 이번 기능 구현을 통해 File API 에 대해 실제 써보는 경험을 하여 유의미했다. 파일 만들기 쏘 이지~~👊🏻
