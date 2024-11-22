import React, { useState } from 'react';
import { Upload, Button, Modal, List, message } from 'antd';
import { InboxOutlined, UploadOutlined } from '@ant-design/icons';
import Papa from 'papaparse';

const { Dragger } = Upload;

const FileUploader = () => {
  const [file, setFile] = useState(null); // Stores the uploaded file
  const [parsedData, setParsedData] = useState([]); // Stores parsed CSV data
  const [isPreviewVisible, setIsPreviewVisible] = useState(false); // Controls preview modal visibility
  const [isPreviewEnabled, setIsPreviewEnabled] = useState(false); // Enables Preview and Send buttons

  // Handle file selection
  const handleFileChange = (info) => {
    const uploadedFile = info.file.originFileObj;
    if (uploadedFile.type !== 'text/csv') {
      message.error('Only CSV files are allowed!');
      return;
    }
    setFile(uploadedFile);
    setIsPreviewEnabled(true); // Enable Preview and Send buttons
    message.success('File selected successfully.');
  };

  // Parse CSV data using PapaParse
  const parseFile = () => {
    if (!file) {
      message.warning('Please upload a file first!');
      return;
    }

    Papa.parse(file, {
      header: false,
      skipEmptyLines: true,
      complete: (result) => {
        const data = result.data.map((row) => row[0]); // Extract single-column data
        setParsedData(data);
        setIsPreviewVisible(true); // Open modal with parsed data
      },
      error: () => {
        message.error('Failed to parse the CSV file.');
      },
    });
  };

  // Send the file to the Spring Boot backend
  const sendFileToBackend = async () => {
    if (!file) {
      message.warning('Please upload a file first!');
      return;
    }

    const formData = new FormData();
    formData.append('file', file); // Add the file to FormData

    try {
      const response = await fetch('http://localhost:8080/api/upload', {
        method: 'POST',
        body: formData,
      });

      if (!response.ok) {
        throw new Error(`Error: ${response.statusText}`);
      }

      const result = await response.json();
      message.success('File uploaded successfully!');
      console.log('Backend Response:', result);
    } catch (error) {
      message.error(`Failed to upload the file: ${error.message}`);
    }
  };

  return (
    <div style={{ padding: '20px' }}>
      <Dragger
        accept=".csv"
        beforeUpload={() => false} // Prevent automatic upload
        onChange={handleFileChange}
        style={{ padding: '20px', borderRadius: '10px' }}
      >
        <p className="ant-upload-drag-icon">
          <InboxOutlined />
        </p>
        <p className="ant-upload-text">Click or drag a CSV file to this area to upload</p>
        <p className="ant-upload-hint">Only single CSV file uploads are supported.</p>
      </Dragger>

      <div style={{ marginTop: '20px' }}>
        <Upload
          accept=".csv"
          beforeUpload={() => false} // Prevent automatic upload
          onChange={handleFileChange}
          showUploadList={false} // Hide the upload list
        >
          <Button icon={<UploadOutlined />} style={{ marginRight: '10px' }}>
            Select File
          </Button>
        </Upload>

        <Button
          type="primary"
          disabled={!isPreviewEnabled}
          onClick={parseFile}
          style={{ marginRight: '10px' }}
        >
          Preview
        </Button>

        <Button type="primary" disabled={!isPreviewEnabled} onClick={sendFileToBackend}>
          Send to Backend
        </Button>
      </div>

      <Modal
        title="CSV Data Preview"
        visible={isPreviewVisible}
        onCancel={() => setIsPreviewVisible(false)}
        footer={[
          <Button key="close" onClick={() => setIsPreviewVisible(false)}>
            Close
          </Button>,
        ]}
      >
        <List
          bordered
          dataSource={parsedData}
          renderItem={(item) => <List.Item>{item}</List.Item>}
        />
      </Modal>
    </div>
  );
};

export default FileUploader;