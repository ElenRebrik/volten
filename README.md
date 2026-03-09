/**
 * @license
 * SPDX-License-Identifier: Apache-2.0
 */

import React, { useState } from 'react';
import { LetterForm } from './components/LetterForm';
import { LetterPreview } from './components/LetterPreview';
import { LetterData, DEFAULT_TEMPLATE } from './types';
import { exportToDocx } from './utils/docxExport';
import { convertPdfToImage } from './utils/pdfToImage';
import { generateDefaultLogo } from './utils/logoGenerator';
import { renderTemplate } from './utils/templateUtils';
import { FileText, Download, Trash2 } from 'lucide-react';

const initialData: LetterData = {
  counterparty: '',
  position: 'Директору',
  addresseeName: '',
  contractNumber: '',
  contractDate: '',
  deliveryDate: '',
  equipmentName: '',
  debtAmount: '',
  outgoingNumber: '',
  letterDate: '',
  signatoryName: 'Пашко Є. В.',
  signatoryPosition: 'директор ТОВ "НВП ВОЛЬТЕН"',
  additionalText: '',
};

export default function App() {
  const [data, setData] = useState<LetterData>(initialData);
  const [bodyText, setBodyText] = useState(DEFAULT_TEMPLATE);
  const [customText, setCustomText] = useState<string | null>(null);
  const [logoUrl, setLogoUrl] = useState<string | undefined>();

  React.useEffect(() => {
    generateDefaultLogo().then(setLogoUrl);
  }, []);

  const handleClear = () => {
    setData(initialData);
    setBodyText(DEFAULT_TEMPLATE);
    setCustomText(null);
    generateDefaultLogo().then(setLogoUrl);
  };

  const handleDataChange = (newData: LetterData) => {
    setData(newData);
    setCustomText(null); // Reset custom text when form changes so template updates
  };

  const handleDownload = async () => {
    try {
      const finalBodyText = customText !== null ? customText : renderTemplate(bodyText, data);
      await exportToDocx(data, finalBodyText, logoUrl);
    } catch (error) {
      console.error('Error exporting DOCX:', error);
      alert('Помилка при генерації файлу Word.');
    }
  };

  const handleUploadLogo = (e: React.ChangeEvent<HTMLInputElement>) => {
    const file = e.target.files?.[0];
    if (!file) return;
    const url = URL.createObjectURL(file);
    setLogoUrl(url);
  };

  return (
    <div className="min-h-screen bg-gray-50 text-gray-900 font-sans">
      <header className="bg-white border-b border-gray-200 px-6 py-4 sticky top-0 z-10 shadow-sm">
        <div className="max-w-7xl mx-auto flex items-center justify-between">
          <div className="flex items-center gap-3">
            <div className="bg-blue-600 p-2 rounded-lg">
              <FileText className="w-6 h-6 text-white" />
            </div>
            <div>
              <h1 className="text-xl font-bold text-gray-900">Генератор листів-вимог</h1>
              <p className="text-sm text-gray-500">ТОВ «НВП «ВОЛЬТЕН»</p>
            </div>
          </div>
          <div className="flex gap-3">
            <button
              onClick={handleClear}
              className="flex items-center gap-2 px-4 py-2 text-sm font-medium text-gray-700 bg-white border border-gray-300 rounded-lg hover:bg-gray-50 transition-colors"
            >
              <Trash2 className="w-4 h-4" />
              Очистити
            </button>
            <button
              onClick={handleDownload}
              className="flex items-center gap-2 px-4 py-2 text-sm font-medium text-white bg-emerald-600 rounded-lg hover:bg-emerald-700 transition-colors shadow-sm"
            >
              <Download className="w-4 h-4" />
              Завантажити Word
            </button>
          </div>
        </div>
      </header>

      <main className="max-w-7xl mx-auto p-6 grid grid-cols-1 lg:grid-cols-12 gap-6">
        {/* Left Column: Form */}
        <div className="lg:col-span-5 flex flex-col gap-6">
          <LetterForm
            data={data}
            onChange={handleDataChange}
            onClear={handleClear}
            onGenerate={() => {}}
            onDownload={handleDownload}
            onUploadLogo={handleUploadLogo}
          />
        </div>

        {/* Right Column: Preview */}
        <div className="lg:col-span-7 flex flex-col bg-white rounded-xl shadow-sm border border-gray-100 overflow-hidden h-[calc(100vh-8rem)]">
          <div className="flex border-b border-gray-200 bg-gray-50">
            <div className="flex-1 py-3 px-4 text-sm font-medium text-blue-600 bg-white border-b-2 border-blue-600">
              Прев'ю документа (можна редагувати текст)
            </div>
          </div>

          <div className="flex-1 overflow-hidden">
            <LetterPreview
              data={data}
              bodyText={bodyText}
              customText={customText}
              onCustomTextChange={setCustomText}
              logoUrl={logoUrl}
            />
          </div>
        </div>
      </main>
    </div>
  );
}

